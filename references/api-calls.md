# API Calls Reference (v2.1.1)

All API call patterns for the SuperClaw Solana Scout. Routes through the SuperClaw Scout Proxy.

## Proxy base URL

```bash
source config/proxy.env
# $SCOUT_PROXY_URL is now available
```

## BYOK override

If `config/keys.env` has non-empty values, call Birdeye/Helius directly. Otherwise use proxy.

## Critical execution notes

- Use `curl` via shell exec. `web_fetch` may strip headers.
- For getProgramAccounts responses (can be 50-200MB), pipe through `jq` to count. Do NOT load full response into memory.
- Helius RPC: 200ms minimum delay between calls, sequential.

---

## Auth check (Step 1)

```bash
curl -s "$SCOUT_PROXY_URL/birdeye/defi/price?address=So11111111111111111111111111111111111111112"
curl -s -X POST "$SCOUT_PROXY_URL/helius" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getHealth"}'
```

Expected Birdeye: `{"success":true,"data":{...}}`
Expected Helius: `{"jsonrpc":"2.0","result":"ok","id":1}`

On failure: STOP with `❌ Scout proxy unreachable. Retry in a few minutes.`

---

## Trending tokens discovery (Step 2)

```bash
curl -s "$SCOUT_PROXY_URL/birdeye/defi/tokenlist?sort_by=v24hUSD&sort_type=desc&offset=0&limit=30"
```

Fetch 30. After exclusions from `config/exclusions.json`, take top 15 unique candidates.

If 24h Δ% is null for any token:
```bash
curl -s "$SCOUT_PROXY_URL/birdeye/defi/price?address=<MINT>&include_liquidity=true"
```

---

## Per-token analysis (Step 3)

For each of 15 tokens, sequentially with 200ms delay between Helius calls:

### (a) getTokenSupply
```bash
-d '{"jsonrpc":"2.0","id":1,"method":"getTokenSupply","params":["<MINT>"]}'
```

### (b) getAccountInfo (mint authority + program ID)
```bash
-d '{"jsonrpc":"2.0","id":1,"method":"getAccountInfo","params":["<MINT>",{"encoding":"jsonParsed"}]}'
```

Capture:
- `result.value.data.parsed.info.mintAuthority` — null = revoked, anything else = open mint
- `result.value.owner` — program ID:
  - `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` = legacy SPL
  - `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb` = Token-2022

### (c) getTokenLargestAccounts
```bash
-d '{"jsonrpc":"2.0","id":1,"method":"getTokenLargestAccounts","params":["<MINT>"]}'
```

Sum top 10 uiAmount / total supply = top-10 concentration %.

### (d) Holder count via jq streaming — CRITICAL

getProgramAccounts can return 50-200MB responses (100k-700k+ holders for popular tokens). Loading into memory returns 0 or crashes. ALWAYS pipe through jq to count.

**Legacy SPL:**
```bash
curl -s -X POST "$SCOUT_PROXY_URL/helius" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getProgramAccounts","params":["TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",{"encoding":"jsonParsed","filters":[{"dataSize":165},{"memcmp":{"offset":0,"bytes":"<MINT>"}}]}]}' \
  | jq '.result | length'
```

**Token-2022 (NO dataSize filter):**
```bash
curl -s -X POST "$SCOUT_PROXY_URL/helius" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getProgramAccounts","params":["TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb",{"encoding":"jsonParsed","filters":[{"memcmp":{"offset":0,"bytes":"<MINT>"}}]}]}' \
  | jq '.result | length'
```

jq output is a single integer. Use directly. If null/empty: check response with `curl ... | head -c 200` to see error. Mark "unverified" only if jq actually fails.

To filter to only organic holders (uiAmount > 0):
```bash
curl ... | jq '[.result[] | select(.account.data.parsed.info.tokenAmount.uiAmount > 0)] | length'
```

### (e) Top traders

```bash
curl -s "$SCOUT_PROXY_URL/birdeye/defi/v2/tokens/top_traders?address=<MINT>&time_frame=24h&sort_type=desc&sort_by=volume&limit=10"
```

**Response shape (do NOT guess):**
```json
{"success":true,"data":{"items":[{...}]}}
```

Each item's fields:
- `owner` — trader's wallet address (use THIS for cross-token matching, NOT "address")
- `trade`, `tradeBuy`, `tradeSell` — trade counts
- `volume`, `volumeUsd` — volumes
- `totalPnl`, `realizedPnl`, `unrealizedPnl` — P&L in USD
- `tokenAddress` — the token (same for all items)

### Rate limit handling

- 429: wait 2s, retry once
- If retry fails: mark check "unverified", continue
- If 5+ tokens fail entirely: abort with rate-limit message

---

## Accumulation candles (Step 5)

Progressive fallback. Use FIRST attempt that returns ≥5 candles:

**A) 4h × 15m (most precise):**
```bash
curl -s "$SCOUT_PROXY_URL/birdeye/defi/ohlcv?address=<MINT>&type=15m&time_from=<now-14400>&time_to=<now>"
```

**B) 12h × 1h (medium):**
```bash
curl -s "$SCOUT_PROXY_URL/birdeye/defi/ohlcv?address=<MINT>&type=1H&time_from=<now-43200>&time_to=<now>"
```

**C) 24h × 1h (low):**
```bash
curl -s "$SCOUT_PROXY_URL/birdeye/defi/ohlcv?address=<MINT>&type=1H&time_from=<now-86400>&time_to=<now>"
```

If all three return <5 candles: mark "no candle data" and exclude from tiering.

In execution status, note which window was used: ✅ 4h / ⚠️ 12h fallback / ⚠️ 24h fallback / ❌ no data.

---

## Direct mode (BYOK fallback)

If user has own keys, replace `$SCOUT_PROXY_URL/birdeye/<path>` with `https://public-api.birdeye.so/<path>` and add `-H "x-chain: solana" -H "X-API-KEY: $BIRDEYE_API_KEY"`. Replace `$SCOUT_PROXY_URL/helius` with `https://mainnet.helius-rpc.com/?api-key=$HELIUS_API_KEY`. All paths, bodies, and response handling identical.

---

## Error codes

| HTTP | Meaning | Action |
|------|---------|--------|
| 200 | Success | Proceed |
| 400 | Bad request | Check body |
| 403 | Path not allowed | Should not happen |
| 429 | Rate limited | Wait 2s, retry, then unverified |
| 502 | Upstream failed | Same as 429 |
| 503 | Proxy down | Retry, then unverified |
