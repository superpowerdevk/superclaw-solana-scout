# Output Template (v2.1.1)

The exact format for the alpha brief. This is brand-critical — do NOT deviate from the visual structure, emoji choices, or section ordering. Read at Step 6.

## Critical: Visual handoff

Before outputting the final brief, output `═══════════════════════════════════════` to signal the transition from agent thinking to deliverable. Do NOT include analysis monologue or prep talk in the user-facing brief.

## Required output order

1. Divider (handoff signal)
2. # H1 SCOUT REPORT header with timestamp
3. **LOOSE MODE warning (loose mode only)**
4. ## 💎 BOTTOM LINE (v2.1.1 reframed scenarios)
5. Next action line
6. ## 📋 SCAN SUMMARY
7. ## 🎯 CANDIDATES (cards, or scarcity message)
8. ## ⚠️ SYSTEMIC NOTE (only if cross-token pattern detected)
9. ## 📊 EXECUTION STATUS
10. Disclaimer line

Use real markdown headers — `#`, `##`, `###` — they render large bold in chat UIs.

---

## 1. Header

```
═══════════════════════════════════════

# 🔍 SCOUT REPORT

**[YYYY-MM-DD HH:MM UTC] · Solana ecosystem**

═══════════════════════════════════════
```

---

## 2. LOOSE MODE warning (loose only)

```
⚠️ **LOOSE MODE — weaker filtering applied. Higher candidate count, higher risk per pick.**
```

STRICT MODE: omit entirely.

---

## 3. BOTTOM LINE — v2.1.1 Shielded Scenarios

Before outputting, compute these stats:
- TOTAL_DISCOVERED (typically 15)
- TIER_1_COUNT, TIER_2_COUNT
- FAILED_FILTER_COUNT (filtered before tiering)
- BOT_WALLETS (wallet clusters from Step 6a)
- BOT_TOKENS (unique tokens the cluster touched)

```
## 💎 BOTTOM LINE

**[scenario one-liner]**
```

### Six scenarios

**A — 3+ Tier 1 picks (any mode):**
> `🎯 3 clean alpha picks found — strongest is $[TICKER] ([+X% 24h], full accumulation).`

**B — 1-2 Tier 1 picks (any mode):**
> `🎯 [N] Tier 1 pick(s) + [M] watchlist near-miss(es). Strongest: $[TICKER].`

**C — 0 Tier 1, 1+ Tier 2, NO bot cluster:**
> `⚠️ No clean alpha. [N] watchlist near-miss(es) — partial signals only. Re-run in 2-4h.`

**D — 0 Tier 1, 1+ Tier 2, bot cluster detected:**
> `🛡️ [N] watchlist near-miss(es) survived screening — but Scout caught [BOT_WALLETS] coordinated wallets running [BOT_TOKENS] of [TOTAL_DISCOVERED] trending tokens. Treat near-misses with caution.`

**E — 0/0, bot cluster detected:**
> `🛡️ Scout shielded you from a [BOT_WALLETS]-wallet bot farm running [BOT_TOKENS] of [TOTAL_DISCOVERED] trending tokens. **Filter pass rate: 0 of [TOTAL_DISCOVERED].** No real alpha to act on. Re-run in 2-4h when this operator rotates.`

**F — 0/0, other issues (concentration, open mints, no bot cluster):**
> `🛡️ Scout filtered [FAILED_FILTER_COUNT] of [TOTAL_DISCOVERED] trending tokens — dominated by [pattern, e.g. "extreme holder concentration" or "open mint authorities"]. Better to skip than force a trade. Re-run in 2-4h.`

The 🛡️ emoji reframes "no alpha" outputs as defensive wins, not failures. The 🎯 emoji is for actual alpha-found scenarios.

---

## 4. Next action

```
⏭️ **[next action]**
```

Examples:
- `⏭️ **Re-scan in 2-4h**`
- `⏭️ **Watch $WIF for breakout above $0.85**`
- `⏭️ **Skip this cycle — try again at next trending refresh**`

---

## 5. SCAN SUMMARY

```
## 📋 SCAN SUMMARY

**Survivors after filters:**
- $[TICKER]: Tier [N] ([brief reason])
- [Excluded tickers]: [reason]

**Cross-token patterns:**
- [Pattern 1 — name wallets if applicable]
- [Pattern 2]
- If none: "None detected — trending list looks organic."
```

---

## 6. CANDIDATES

### Scenarios

- **3+ Tier 1:** `## 🎯 CANDIDATES` then 🥇🥈🥉 cards
- **1-2 Tier 1:** Tier 1 cards as 🥇 (or 🥇🥈), then `### ⚠️ Watchlist near-misses` then 👀 cards
- **0 Tier 1, 1+ Tier 2:** Replace header with `## ⚠️ WATCHLIST (no Tier 1 matches)` then bold warning then 👀 cards
- **0 / 0:** SKIP candidates section entirely. Go to systemic note + execution status.

### Card format (EXACT)

```
### [🥇/🥈/🥉/👀] $[TICKER] · [Name]  |  [🟢 Tier 1 / 🟡 Tier 2]

**[Δ emoji] [±X.XX%] 24h**

💰 **MC** $[X]  ·  📊 **Vol** $[Y]  ·  👥 **Holders** [N]

**🚩 RISKS**
- [icon] [3-7 word verdict]
- [icon] [3-7 word verdict]
- [icon] [3-7 word verdict]

**✨ OPPORTUNITY**
[1 sentence max]

**🎯 ENTRY ZONE**
$[X] – $[Y]

**🔗 Chart:** [dexscreener.com/solana/[first_8_chars]...](https://dexscreener.com/solana/[FULL_MINT])

────────
```

### Δ emoji map

| Δ% | Emoji |
|----|-------|
| ≥ +20% | 🟢 |
| +5 to +20 | 🔼 |
| -5 to +5 | ➖ |
| -5 to -20 | 🔽 |
| ≤ -20% | 🔻 |

### Risk icons

| Icon | Risk type |
|------|-----------|
| 🌀 | wash trading |
| 🪂 | falling knife |
| ❓ | data missing |
| 🤖 | bot cluster / coordinated launch / T-2022 farming |
| 🐋 | whale concentration |
| 🔓 | open mint authority |
| 💀 | honeypot / scam pattern |
| 📉 | declining momentum |
| 🆕 | unproven launch |
| 🎭 | anon team |

### Opportunity icons (optional)

| Icon | Type |
|------|------|
| 🚀 | breakout setup |
| 🏗️ | clean accumulation |
| 🔥 | organic volume |
| 💎 | oversold bounce |

---

## 7. SYSTEMIC NOTE (conditional)

ONLY if cross-token patterns detected. Otherwise OMIT.

```
## ⚠️ SYSTEMIC NOTE

[Name actual "owner" wallet addresses (first 8 chars + "..."). Count exactly how many candidates each wallet touched. Describe the pattern in 1-3 sentences. Pure analysis — no marketing framing.]
```

---

## 8. EXECUTION STATUS

ALWAYS at bottom. Markdown table.

```
## 📊 EXECUTION STATUS

| Check | Result |
|-------|--------|
| Token discovery | ✅ 30→15 post-exclusion |
| MC / Vol / 24h Δ | [✅/⚠️] [note fallback if used] |
| Mint authority | [✅/⚠️/❌] |
| Top 10 holder % | [✅/⚠️/❌] |
| Holder count | [✅/⚠️/❌] |
| Top traders | [✅/⚠️/❌] [add (X/15) if partial] |
| Candles | [✅/⚠️/❌] [note window breakdown] |
```

Status icons:
- ✅ = clean run for all 15 tokens
- ⚠️ = caveats (partial data, fallbacks used)
- ❌ = failed entirely
- ⏭️ = skipped (e.g. no survivors to tier candles for)

---

## 9. Disclaimer

```
═══════════════════════════════════════

*Not financial advice. Execute manually. Your SuperClaw surfaces alpha — you decide.*
```

The italic styling and `═══` divider give the disclaimer a soft visual cap.

---

## Why this format works

- **`═══` heavy dividers** create unmistakable visual borders
- **`#` H1 SCOUT REPORT** is the biggest text in the chat — scannable from across the room
- **`##` H2 section headers** anchor each major section
- **🛡️ shield emoji** reframes "no alpha" as defensive win — users feel protected, not failed
- **🎯 target emoji** signals actual alpha discovery
- **Bold field labels and bold Δ%** make key numbers pop
- **Markdown table** for execution status renders cleanly across mobile and desktop
- **Italic disclaimer** softens the closing without competing with content

This is the canonical visual spec. Lock it.
