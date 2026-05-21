# SuperClaw Solana Alpha Scout

The canonical SuperClaw skill for surfacing real alpha on Solana — not just trending tokens.

## What it does

Scans the top trending Solana tokens, filters out wash-traded micro-caps and risky contracts using on-chain holder, mint authority, and concentration analysis, and surfaces only candidates with confirmed accumulation patterns. Output is a mobile-optimized brief with tier ranking, risk flags, and cross-token bot-cluster detection.

**The difference from naive trending-token tools:**
- Filters out wash-trading and bot-farmed launches *before* surfacing
- Detects coordinated wallet clusters across the trending list (signature feature)
- Honest scarcity: when no clean alpha exists, says so instead of forcing 3 picks

## Quick start

After install, run the scout — no setup needed:

```
Run the Solana alpha scout.
```

That's it. Keys live server-side in the SuperClaw Scout Proxy. Zero configuration.

### Two modes (v2.1)

**Strict mode (default):**
```
Run the Solana alpha scout.
```
5 filters, multi-list discovery, honest scarcity. Surfaces only genuinely clean alpha.

**Loose mode (variant):**
```
Run the Solana alpha scout — loose mode.
```
3 filters only. Surfaces more candidates (with elevated risk per pick). For momentum traders who want signals even when the trending list is partially fake.

### Optional power-user mode

If you want to use your own paid API keys instead of the shared proxy:

```
/scout setup
```

Walks you through getting free-tier Birdeye + Helius keys.

## Installation

### Option 1: User-scope (available across all projects)

```bash
cp -r superclaw-solana-scout ~/.openclaw/skills/
```

### Option 2: Workspace-scope (this project only)

```bash
cp -r superclaw-solana-scout <your-project>/skills/
```

After install, trigger the skill from any agent session by asking for Solana alpha, trending tokens, or running `Run the Solana alpha scout.`

## Using your own keys

The shared dev keys work for testing but hit rate limits at scale. For production use, switch to your own keys:

```
/scout setup
```

The setup flow walks you through getting free-tier keys from Birdeye and Helius (both ~2 minutes to sign up). Your keys are stored locally in `config/keys.env`.

## Output format

Every scout run produces:

1. **💎 BOTTOM LINE** — one-line verdict at the top
2. **⏭️ Next action** — what to do next (re-scan, watch a ticker, etc.)
3. **Candidate cards** — top picks with tier rank, risk flags, opportunity, entry zone
4. **⚠️ Systemic note** — cross-token pattern detection (if applicable)
5. **📊 Execution status** — proof-of-work table

See `references/output-template.md` for the full format spec with examples.

## File structure

```
superclaw-solana-scout/
├── SKILL.md                    # Main entry point
├── README.md                   # This file
├── references/
│   ├── api-calls.md            # Birdeye + Helius endpoint definitions
│   ├── filter-logic.md         # Filter thresholds + tier rules
│   ├── output-template.md      # Exact brief format with examples
│   └── setup-flow.md           # /scout setup interactive flow
└── config/
    ├── keys.env                # API keys (shared dev keys preloaded)
    └── exclusions.json         # Stablecoins/wrappers to filter out
```

## Customization

- **Exclusion list:** edit `config/exclusions.json` to add stablecoins, wrappers, or known scam tokens. Changes take effect on the next scout run.
- **Filter thresholds:** edit `references/filter-logic.md` to adjust top-10 holder cap (default 40%), minimum holder count (default 200), wash-volume ratio (default 10x), etc.
- **Output template:** edit `references/output-template.md` to adjust visual styling, but be aware this is brand-critical — changes affect every scout run users see.

## Brand rules

These are non-negotiable:
- Never force candidates to fill a "top 3" slot. Honest scarcity beats fake alpha.
- Always include the systemic note when cross-token patterns are detected.
- The disclaimer line is exact: `Not financial advice. Execute manually. Your SuperClaw surfaces alpha — you decide.`

## Support

For issues or improvements, contact the SuperClaw team.

---

🤖 Part of the SuperClaw skills ecosystem. The agent economy. Yours starts now.
