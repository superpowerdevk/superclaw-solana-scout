# /scout setup — Interactive Credential Flow

This is the user-facing setup flow for the Solana Alpha Scout. Read this only when:
1. The user explicitly types `/scout setup` or asks to configure their own keys
2. `config/keys.env` is missing or has empty values for `BIRDEYE_API_KEY` or `HELIUS_API_KEY` AND the user has not opted into shared dev keys

The skill ships with shared dev keys preloaded in `config/keys.env` so most users never need this flow. They can run the scout immediately. This flow is for users who:
- Hit rate limits on shared keys
- Want their own key for reliability at scale
- Want to use a paid tier for production usage
- Are running many SuperClaws concurrently

---

## Step 1: Greet and explain

Output to the user:

```
👋 Let's set up your own API keys for the Solana Alpha Scout.

You'll need 2 keys (both have free tiers):

1. Birdeye API key — for token discovery, market data, and candles
   Free tier: ~30 req/min
   Sign up: https://bds.birdeye.so/

2. Helius API key — for Solana on-chain holder and contract data
   Free tier: 100k credits/day, ~10 req/sec
   Sign up: https://dashboard.helius.dev/

Both take ~2 minutes to set up. Once you have them, paste them here and
I'll configure everything for you.
```

Then ask:

```
Do you already have both keys, or do you need help getting them first?
```

Wait for the user's response.

---

## Step 2: If user needs help getting keys

Provide step-by-step instructions:

```
🟦 Birdeye:
1. Go to https://bds.birdeye.so/
2. Sign up (email or wallet)
3. Dashboard → API Keys → Create New Key
4. Copy the key

🟪 Helius:
1. Go to https://dashboard.helius.dev/
2. Sign up (email or wallet)
3. Dashboard → API Keys (one is auto-created on signup)
4. Copy the key

Once you have both, paste them here and I'll handle the rest.
```

Wait for the user to come back with keys.

---

## Step 3: Collect keys

Ask:

```
Paste your keys below in this format:

Birdeye key: <your_birdeye_key>
Helius key: <your_helius_key>
```

Wait for response.

---

## Step 4: Validate keys before writing

Before writing to `config/keys.env`, validate that the keys actually work:

```bash
# Test Birdeye
BIRDEYE_TEST=$(curl -s -w "\nHTTP_CODE:%{http_code}" -H "x-chain: solana" \
  -H "X-API-KEY: <USER_BIRDEYE_KEY>" \
  "https://public-api.birdeye.so/defi/price?address=So11111111111111111111111111111111111111112")

# Test Helius
HELIUS_TEST=$(curl -s -X POST "https://mainnet.helius-rpc.com/?api-key=<USER_HELIUS_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getHealth"}')
```

Check:
- Birdeye returns HTTP 200 with `"success":true`
- Helius returns `"result":"ok"`

### If both pass:

Write the keys to `config/keys.env`:

```bash
cat > config/keys.env << EOF
# SuperClaw Solana Alpha Scout — user-configured keys
# Set by /scout setup on $(date -u +"%Y-%m-%d %H:%M UTC")

BIRDEYE_API_KEY="<USER_BIRDEYE_KEY>"
HELIUS_API_KEY="<USER_HELIUS_KEY>"
EOF
```

Then output:

```
✅ Keys configured and verified.

You're all set. Run the scout anytime with:
  Run the Solana alpha scout.

Your keys are saved in config/keys.env. To switch back to shared dev keys,
delete that file or run /scout setup again.
```

### If either fails:

Tell the user which key failed and offer to retry:

```
❌ [Birdeye / Helius] key validation failed.

Possible causes:
- Key typed incorrectly (check for extra spaces or missing characters)
- Key not yet activated on the provider's side (sometimes takes 1-2 minutes)
- Key revoked or expired

Want to try again? Paste both keys again and I'll re-validate.
```

Do NOT write invalid keys to `config/keys.env`. Loop back to Step 3.

---

## Step 5: Offer to run the scout immediately

After successful setup, offer:

```
Want to run a scout right now to confirm everything works?
```

If yes: proceed to the main scout workflow per SKILL.md.
If no: end the setup flow.

---

## Edge cases

### User pastes keys but in the wrong format
If you can't unambiguously parse which key is Birdeye vs Helius:
- Birdeye keys are typically 32-character hex strings (e.g., `8cfeba044044473696005e809801e16f`)
- Helius keys are typically UUID format with hyphens (e.g., `ed005ccd-4a8f-47cd-a03f-3880d287879a`)

If the user pasted in ambiguous format, ask them to clarify:
```
I want to make sure I assign these correctly. Which one is Birdeye and which is Helius?
```

### User wants to revert to shared dev keys
If user says "use the shared keys" or "revert" or "go back to default":

```bash
# Restore the shared keys
cat > config/keys.env << EOF
# SuperClaw Solana Alpha Scout — shared dev keys (default)
# These keys are shared across all users and may hit rate limits.
# Run /scout setup to use your own keys.

BIRDEYE_API_KEY="<SHARED_BIRDEYE_KEY_FROM_DEFAULTS>"
HELIUS_API_KEY="<SHARED_HELIUS_KEY_FROM_DEFAULTS>"
EOF
```

(The skill ships with these keys preloaded; just restore them.)

### User wants to delete their keys for privacy
If user says "delete my keys" or similar:

```bash
# Wipe the user's config
rm config/keys.env
# Restore default shared keys
cp config/keys.env.default config/keys.env  # if this default file exists
# Otherwise restore from the values shipped with the skill
```

Confirm:
```
✅ Your keys have been removed. Shared dev keys are now active.
```
