---
name: superclaw-solana-scout
description: SuperClaw's canonical Solana alpha scout. Scans top trending Solana tokens across multiple discovery lists (volume, price change, holder growth, liquidity), filters out wash-traded micro-caps and risky contracts using on-chain holder, mint authority, and concentration analysis, and surfaces only candidates with confirmed accumulation patterns. Output is a mobile-optimized brief with tier ranking, risk flags, and cross-token bot-cluster detection. Two modes available — strict (default, brand-canonical) and loose (more candidates surface, weaker filtering). Trigger ANY time the user asks for Solana alpha, trending Solana tokens, alpha picks, trade ideas on Solana, "what's hot on Solana", market scanning, or runs phrases like "scout", "find alpha", "scan solana", "alpha brief", "morning scan" — even if they don't explicitly name the skill.
---

# SuperClaw Solana Alpha Scout

You are running the canonical SuperClaw Solana alpha scout workflow. This is a brand-critical output: the format, tier logic, and honest-scarcity behavior are non-negotiable.

## Zero-setup architecture

This skill uses the SuperClaw Scout Proxy at the URL defined in `config/proxy.env`. API keys live on the proxy server-side — users do NOT need to configure their own keys.

**Optional power-user mode:** If `config/keys.env` exists AND contains non-empty `BIRDEYE_API_KEY` and `HELIUS_API_KEY`, the skill calls Birdeye and Helius directly with those keys instead of routing through the proxy.

## Two scout modes

The user can invoke the skill in two ways:

**STRICT MODE (default, canonical):** "Run the Solana alpha scout" / "Find alpha" / etc.
- 5 filters, full pattern detection, multi-list discovery
- Honest scarcity — outputs "no alpha" when trending list is fake
- Brand-positive — surfaces only genuinely clean candidates

**LOOSE MODE (variant):** "Run the Solana alpha scout — loose mode" / "Quick scout" / "Loose scan"
- 3 filters only (concentration + mint authority + holder count)
- No wash-volume signature filter, no coordinated-bot filter
- Almost always surfaces 3 picks — for users who want momentum plays even with elevated risk
- Output clearly labels mode as "LOOSE MODE — weaker filtering"

Determine mode from user phrasing. If unclear, default to strict.

## Quick orientation

The workflow runs in 6 sequential steps:

1. **Auth/proxy check** — verify proxy or BYOK credentials work
2. **Multi-list discovery** — pull from 4 trending lists, dedupe, exclude stablecoins/wrapped
3. **Per-token analysis** — on-chain holder, mint, top-trader checks
4. **Filter** — apply mode-specific filter set
5. **Accumulation + tiering** — fetch candles, classify Tier 1 / 2 / 3
6. **Output** — render the brief

## Setup — load config

1. Read `config/proxy.env` to load `SCOUT_PROXY_URL`. Default: `https://api.superpower.io/scout`.
2. Check `config/keys.env`:
   - If both `BIRDEYE_API_KEY` and `HELIUS_API_KEY` are set and non-empty → BYOK mode
   - Otherwise → proxy mode (default)
3. Determine scout mode (strict / loose) from user phrasing.
4. Proceed to Step 1.

## Execution rules

- **Use curl, not web_fetch.** Shell exec only.
- **Sequential Helius calls.** 200ms minimum delay between RPC calls.
- **Rate-limit handling.** On 429: wait 2s, retry once. If still failing, mark check "unverified" and continue. If 5+ tokens fail entirely, abort with rate-limit message.
- **Honest scarcity (strict mode).** Never force candidates. If no clean alpha, say so.
- **Loose mode acknowledges trade-offs.** Always label brief as "LOOSE MODE — weaker filtering applied" at the top.
- **Brand consistency.** Output format is non-negotiable.

## Workflow steps

### Step 1: Auth/proxy check
Per `references/api-calls.md` → "Auth check". On failure: STOP with the defined failure message.

### Step 2: Multi-list discovery
Per `references/api-calls.md` → "Multi-list discovery". Pull from 4 trending lists, dedupe, apply exclusions, take top 15 unique candidates.

### Step 3: Per-token analysis
Per `references/api-calls.md` → "Per-token analysis". For each of the 15 tokens, run on-chain checks sequentially with 200ms pacing.

### Step 4: Apply filters
Per `references/filter-logic.md` → "Filters". Filter set depends on mode (strict = 5 filters, loose = 3 filters).

### Step 5: Accumulation + tiering
Per `references/filter-logic.md` → "Accumulation + Tiering". Same tier logic in both modes.

### Step 6: Output the brief
Render EXACTLY per `references/output-template.md`.

**CRITICAL UX rule:** Before outputting the final brief, output the `═══════════════════════════════════════` divider line. Do NOT include analysis monologue, "now let me compile" prep talk, or thinking-out-loud in the user-facing brief.

For LOOSE MODE, add this line immediately after the H1 SCOUT REPORT header:
```
⚠️ **LOOSE MODE — weaker filtering applied. Higher candidate count, higher risk per pick.**
```

The brief uses the v2.1.1 "shielded" framing for no-alpha scenarios — see output-template.md for all six BOTTOM LINE scenarios (A-F). The 🛡️ emoji signals defensive wins (scout protected user from bot farms / extreme concentration), 🎯 signals actual alpha discovery.

The brief includes:
- Divider + H1 SCOUT REPORT header with timestamp
- LOOSE MODE warning (loose only)
- 💎 BOTTOM LINE one-liner (scenario-matched, v2.1.1 reframed)
- Next action line
- 📋 SCAN SUMMARY (inline transparency)
- 🎯 CANDIDATES (cards or scarcity message)
- ⚠️ SYSTEMIC NOTE (only if cross-token patterns detected — pure analysis, no marketing)
- 📊 EXECUTION STATUS (markdown table)
- Italic disclaimer (exact wording)

## Critical brand rules

- **Strict mode: never force candidates.** Honest scarcity beats fake alpha.
- **Loose mode: always label as such.** Users must know they opted into weaker filtering.
- **Systemic note required when cross-token patterns detected.** Both modes.
- **Risk flags are 3-7 word verdicts, not sentences.**
- **Disclaimer line exact:** `Not financial advice. Execute manually. Your SuperClaw surfaces alpha — you decide.`

## Reference files

- `references/api-calls.md` — proxy endpoints + multi-list discovery
- `references/filter-logic.md` — strict vs loose filter sets, tier rules
- `references/output-template.md` — brief format with all scenarios

## Config files

- `config/proxy.env` — proxy URL
- `config/keys.env` — optional BYOK keys
- `config/exclusions.json` — stablecoins/wrappers to filter out
