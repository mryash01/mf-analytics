# DAILY CHECKLIST — v3 (60 Seconds)

Use this as a fast reference while running the skill. Detailed scoring is in
`SIGNAL_FRAMEWORK.md`. If you've set up the Telegram pipeline (`SETUP_TELEGRAM.md`),
the alert script does all of this automatically and pings you only when there's
something to do — this checklist is the manual fallback.

---

## The 6 Data Points

Run web_search for each. Write the number down before scoring.

```
[ ]  Nifty 50         close = _______   52W H = _______   200-DMA = _______   20-DMA = _______
[ ]  Nifty 500        close = _______   52W H = _______
[ ]  Nifty Midcap 150 close = _______   52W H = _______
[ ]  Nifty Smallcap250 close = _______   52W H = _______
[ ]  India VIX        value = _______
[ ]  Trend check      Nifty 50 above 20-DMA in last 3 days? Y/N
                      Making higher lows this week vs last? Y/N
```

## The 3 Drawdowns

```
Nifty 500       DD = (close - 52WH) / 52WH × 100 = _______ %
Nifty Midcap150 DD = (close - 52WH) / 52WH × 100 = _______ %
Nifty Smallcap250 DD = (close - 52WH) / 52WH × 100 = _______ %
```

## Quick Score Lookup (v3)

Nifty 500 drawdown: -3→1pt | -7→2pt | -10→3pt | -13→4pt | -17→5pt
Midcap 150 drawdown: -5→1pt | -10→2pt | -14→3pt | -18→4pt | -23→5pt
Smallcap 250 drawdown: -7→1pt | -12→2pt | -17→3pt | -22→4pt | -28→5pt

200-DMA (Nifty 50): Below by 0-3% = 1pt | Below by 3%+ = 2pt

**Subtract 2 if Nifty 50 below 200-DMA for >60 trading days — UNLESS Dim1=5 AND Dim5=1
(v3 exception: capitulation + discrete shock = real bottom, no penalty).**

VIX: 18-22 = 0.5 | 22-28 = 1 | >28 = 1.5

20-day fall (Nifty 500): >5% = 0.5 | >10% = 1
20-day fall (Midcap): >7% = 0.5 | >13% = 1
20-day fall (Smallcap): >9% = 0.5 | >16% = 1

Discrete shock in past 7 days + DD ≥-7%: 1pt
(slow grinds / FII selling / "tariff fears" / "earnings concerns" do NOT count)

## 🚦 THE STABILIZATION GATE (v2 — HARD)

```
IF Dim 1 ≥ 4 (i.e., big drawdown):
    Did Nifty 50 close above 20-DMA in past 3 days?  Y / N
    Did Nifty 50 make a higher low this week vs last? Y / N
    
    Either Y → GATE CLEARS, use full score
    Both N → CAP score at 6 (Light Buy max)
```

## Decision Cutoffs (v2)

```
Score 0-5: NO BUY. Continue SIP. Do nothing.
Score 6: LIGHT BUY. 15% of reserve.
Score 7: BUY. 25% of reserve.
Score 8: BUY. 40% of reserve.
Score 9-10: STRONG BUY. 50-60% of reserve. Stagger across 3-5 days.
```

## Per-Fund Routing

Each fund's deploy decision uses the score of its benchmark, defined in `funds.json`:

```
Nifty 500 score    → all funds tagged benchmark: nifty500    (Flexi Cap by default)
Midcap 150 score   → all funds tagged benchmark: midcap150   (Mid Cap)
Smallcap 250 score → all funds tagged benchmark: smallcap250 (Small Cap)
```

To see the current routing, open `funds.json`. The Telegram alert lists every fund
mapped to a benchmark that scored ≥ 6.

## Pre-Deploy Reality Check (v2)

Before placing the order, ask:
1. Is SIP running? (Required — never skip SIP for tactical buy.)
2. Was my last deploy on this same drawdown cycle within the past 30 days? (If yes → blackout active. Only re-deploy if drawdown has deepened by ≥5 percentage points.)
3. Am I deploying from a pre-set tactical reserve, not from emergency funds? (Critical.)
4. If markets fall another 10% from here, will I have dry powder? (Yes = OK. No = halve the deploy.)
5. Did the stabilization gate (Dim 6) clear? (Skip this only if Dim 1 < 4.)

If all 5 are clear, execute.

---

## When to Override the Signal

The signal is rule-based. Override only on:

- **Macro red flag the rubric doesn't capture:** banking crisis, sovereign credit event,
  India-specific governance shock. → Halve the deploy size and wait 1 week.
- **Personal cashflow change:** lost job, large unplanned expense. → Pause all tactical buys.
- **Fund-specific news:** AMC fraud, key manager exit, mandate change. → Drop that fund
  from the rotation, redirect to peer in same category.

Do NOT override on:
- Headlines that "feel scary"
- Other analysts being bearish
- A friend recommending you wait
- Your own gut feeling that "it'll fall more"

If the signal says BUY and rubric is intact, BUY. The whole point of having a rule is
removing emotion from the decision.

## Common Mistakes Eliminated in v2/v3

- ❌ v1 "Light Buy" at score 5 → too many false positives. v2+ requires score 6+.
- ❌ v1 fired buy signals mid-decline → v2 stabilization gate blocks deep-drawdown deploys until index actually stops falling.
- ❌ v1 averaged down on slow grinds → v2 30-day blackout + 5% deeper requirement prevents this.
- ❌ v1 awarded event points for general fog → v2 requires a dateable, discrete shock.
- ❌ **v2 under-deployed at the deepest bottoms (Jun 2022, Apr 2025) due to structural bear penalty → v3 makes the penalty conditional: it does NOT apply when Dim1=5 AND Dim5=1 (the capitulation+shock signature of a real bottom).**
