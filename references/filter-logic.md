# Filter Logic & Tier Rules (v2.1)

Filter thresholds, mode-specific filter sets, and tier definitions. Read at Step 4 (filtering) and Step 5 (tiering).

## Filter sets by mode

### STRICT MODE (default canonical) — apply ALL 5 filters

Filter OUT any token where:

1. **Concentration:** Top 10 holders > 40% of supply
2. **Open mint:** mintAuthority is NOT null
3. **Low holders:** Fewer than 200 holders
4. **Wash signature:** 24h vol > 10x MC AND Δ% is flat/null
5. **Bot cluster:** ALL 10 top traders show net negative P&L

### LOOSE MODE (variant) — apply only filters 1-3

Filter OUT any token where:

1. **Concentration:** Top 10 holders > 40% of supply
2. **Open mint:** mintAuthority is NOT null
3. **Low holders:** Fewer than 200 holders

Loose mode SKIPS filters 4 and 5. This is by design — these are the filters that most often eliminate momentum plays (which often have high vol/MC ratios and bot-driven top-trader lists). Users opting into loose mode are accepting that elevated wash/bot exposure.

## Filter rationale

### Filter 1: Concentration
- Threshold: top 10 holders > 40%
- Why: one entity can dump and crash

### Filter 2: Open mint
- Why: open mint can inflate supply, dilute holders

### Filter 3: Low holders
- Threshold: < 200
- Why: thin organic interest, easy to manipulate

### Filter 4: Wash signature (STRICT ONLY)
- Threshold: vol/MC > 10x AND abs(Δ%) < 5
- Why: massive volume with no price movement = wash trading

### Filter 5: Bot cluster (STRICT ONLY)
- Why: legit trading produces winners and losers. All-negative = bot churning

## Unverified data handling

If a filter check has missing data (e.g., getTokenLargestAccounts rate-limited):
- DO NOT auto-filter the token out
- Keep in pool, mark which filters were unverified
- User sees the gap and can discount accordingly

---

## Accumulation + Tiering (Step 5)

For surviving tokens, fetch candles and analyze for TWO patterns.

### Pattern 1: Volume rising
Split candle array into thirds. Pattern confirmed if:
`avg_last_third_volume >= 1.2 * avg_first_third_volume`

### Pattern 2: Higher lows
At least 3 consecutive higher pullback bottoms in the window.

### Tier assignment

| Tier | Vol rising | Higher lows | Mode A action | Mode B action |
|------|-----------|------------|---------------|---------------|
| **1** | ✅ | ✅ | 🥇🥈🥉 top 3 | 🥇🥈🥉 top 3 |
| **2** | ❌ | ✅ | 👀 watchlist | 👀 watchlist |
| **3** | ✅ | ❌ | EXCLUDE | EXCLUDE |
| **3** | ❌ | ❌ | EXCLUDE | EXCLUDE |

### Multi-list discovery bonus

Tokens appearing in 2+ discovery lists (volume + price + liquidity + trade count) get a `+1 multi-list confirmation` flag in their opportunity section. This isn't a tier promotion — it's informational, surfacing to users that the discovery wasn't a single-vector signal.

Example opportunity text: `🚀 Strong breakout setup with multi-list confirmation (discovered via volume + liquidity + trade count)`

### Candle data unavailable

No candle data = NOT included in Tier 1 or 2. Mark "no candle data" in execution status.

---

## Cross-token pattern detection (for Systemic Note)

After per-token analysis, check across all 15 candidates:

### Shared wallet clusters
Same wallets in top_traders of 3+ tokens with similar trade-count/PnL profiles = coordinated bot cluster.

### Identical holder distribution shapes
Multiple Token-2022 launches with suspiciously uniform top-10 distributions = bot-farmed launches.

### Include Systemic Note in output if pattern detected
- Name wallet addresses (first 8 chars + `...`)
- Count tokens affected
- Describe in 1-3 sentences

If no pattern, OMIT this section.

---

## Scenarios + BOTTOM LINE wording

| Tier 1 | Tier 2 | Mode | Wording |
|--------|--------|------|---------|
| 3+ | any | both | `3 clean alpha picks — strongest is $[TICKER] ([+X% 24h], full accumulation).` |
| 1-2 | any | both | `[N] Tier 1 pick(s) + [M] watchlist near-miss(es). Strongest: $[TICKER].` |
| 0 | 1+ | strict | `No clean alpha. [N] watchlist near-miss(es) — [pattern]. Re-run in 2-4h.` |
| 0 | 1+ | loose | `[N] loose-mode picks surfaced — caveat: weaker filters applied. Strongest: $[TICKER].` |
| 0 | 0 | strict | `No alpha matches. Trending dominated by [pattern]. Re-run in 2-4h.` |
| 0 | 0 | loose | `Even loose-mode filters found no picks. Trending list is broken right now. Re-run in 2-4h.` |
