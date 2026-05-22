# SIGNAL FRAMEWORK — v3

The signal is a **sum of points across 5 dimensions plus 1 hard filter**, capped at 10.
Each benchmark (Nifty 500, Midcap 150, Smallcap 250) is scored independently.

> **What changed from v2 to v3:** Re-running v2 on a 5-year backtest (May 2021 → May
> 2026) revealed v2's structural-bear penalty under-deploys at the deepest, best
> bottoms. v3 makes the penalty **conditional**: it no longer applies when the score
> shows both maximum drawdown (Dim 1 = 5) AND a discrete shock event (Dim 5 = 1) —
> the signature of genuine capitulation. This change restores Strong Buy aggression
> at real bottoms (June 2022, April 2025) without re-introducing v1's premature-buy
> failure mode. See `CHANGELOG.md` for details.

---

## Dimension 1 — Drawdown from 52-Week High (max 5 points)

Unchanged from v2 — this is the dominant signal.

### NIFTY 500 (Flexi Cap funds)

| Drawdown from 52W High | Points |
|---|---|
| 0% to -3%   | 0 |
| -3% to -7%  | 1 |
| -7% to -10% | 2 |
| -10% to -13%| 3 |
| -13% to -17%| 4 |
| -17% or worse | 5 |

### NIFTY MIDCAP 150 (Mid Cap funds)

| Drawdown from 52W High | Points |
|---|---|
| 0% to -5%    | 0 |
| -5% to -10%  | 1 |
| -10% to -14% | 2 |
| -14% to -18% | 3 |
| -18% to -23% | 4 |
| -23% or worse| 5 |

### NIFTY SMALLCAP 250 (Small Cap fund)

| Drawdown from 52W High | Points |
|---|---|
| 0% to -7%    | 0 |
| -7% to -12%  | 1 |
| -12% to -17% | 2 |
| -17% to -22% | 3 |
| -22% to -28% | 4 |
| -28% or worse| 5 |

---

## Dimension 2 — Position vs 200-DMA (max 2 points)

Unchanged. Use Nifty 50's 200-DMA as the macro filter for ALL three benchmarks.

| Nifty 50 vs 200-DMA | Points |
|---|---|
| Above 200-DMA by >5%  | 0 |
| Above 200-DMA by 0–5% | 0 |
| Below 200-DMA by 0–3% | 1 |
| Below 200-DMA by 3–8% | 2 |
| Below 200-DMA by >8%  | 2 (capped) |

---

## Dimension 3 — India VIX (max 1.5 points)

Unchanged from v2.

| India VIX Level | Points |
|---|---|
| < 13     | 0 |
| 13 – 18  | 0 |
| 18 – 22  | 0.5 |
| 22 – 28  | 1 |
| > 28     | 1.5 |

---

## Dimension 4 — Velocity of Fall (max 1 point)

Unchanged from v2 — 20-day fall measurement.

| Benchmark | 20-day fall threshold | Points |
|---|---|---|
| Nifty 500 | > 5%   | 0.5 |
| Nifty 500 | > 10%  | 1   |
| Midcap 150 | > 7%  | 0.5 |
| Midcap 150 | > 13% | 1   |
| Smallcap 250 | > 9%| 0.5 |
| Smallcap 250 | > 16%| 1  |

---

## Dimension 5 — Discrete Shock Event (max 1 point)

Unchanged from v2 — strict definition required.

Award **+1 point** ONLY if ALL three are true:
- Drawdown ≥ -7% (Dim 1 already at 2+)
- The event happened in the past 7 trading days
- The event is **discrete and concrete** (war start, surprise rate move, election panic, bank failure, specific tariff announcement day, currency crisis)

Does NOT count: "FII selling continues", "weak earnings season", "valuation concerns",
"tariff fears" (the announcement day counts; weeks of fear do not), "crude oil rising",
"global slowdown worries".

---

## Dimension 6 — Stabilization Filter (HARD GATE)

Unchanged from v2.

Triggers when **Dim 1 ≥ 4 points**. The index must clear one of these to pass:
- Nifty 50 closed above its 20-DMA in the past 3 trading days, OR
- Nifty 50 made a higher low (today's low > the low 5–10 days ago)

If neither is true → cap the score at 6 (Light Buy max).

---

## Final Score Calculation — v3 ⚙️ STRUCTURAL BEAR NOW CONDITIONAL

```
RAW = Dim1 + Dim2 + Dim3 + Dim4 + Dim5
RAW = min(RAW, 10)

IF Dim1 ≥ 4 AND Dim6 stabilization filter NOT cleared:
    FINAL = min(RAW, 6)    # cap at Light Buy until index stabilizes
ELSE:
    FINAL = RAW

# v3 change: structural-bear penalty is now CONDITIONAL
IF Nifty 50 has been below 200-DMA for >60 trading days:
    IF Dim1 == 5 AND Dim5 == 1:
        # Capitulation + discrete shock = real bottom signature, NO penalty
        pass
    ELSE:
        FINAL = FINAL - 2  # structural bear penalty applies
```

Round 0.5s UP only when Dim 1 ≥ 3.

### Why the v3 Exception Works

The structural-bear penalty exists to prevent BUY signals firing in the middle of
extended bears (where v1 over-fired in 2022 and Mar 2025). At those mid-decline
moments, Dim 1 is typically 3-4 (not maxed) and Dim 5 is typically 0 (no discrete
event — just "things are bad"). The penalty correctly fires there.

At true capitulation bottoms — June 2022 (Fed 75bp surprise), April 2025 (Trump
Liberation Day tariffs) — Dim 1 hits 5 AND Dim 5 hits 1. This combination is rare
(3 events in 5 years across 1,235 trading days) and is the actual signature you
want to deploy aggressively into. v3's exception preserves the mid-decline brake
while restoring full conviction at the bottom.

---

## Action Mapping — v3

Unchanged from v2.

| Score | Action | Deploy % of Tactical Reserve |
|-------|--------|------------------------------|
| 0–5   | NO BUY — wait | 0% |
| 6     | LIGHT BUY — small toe in | 15% |
| 7     | BUY — solid signal | 25% |
| 8     | BUY — high conviction | 40% |
| 9     | STRONG BUY — generational entry | 50% |
| 10    | STRONG BUY — crisis dislocation | 60% |

**30-day blackout rule retained from v2.** After any deploy, no re-deploy on the same
drawdown cycle for 30 days unless drawdown deepens by ≥5 percentage points from your
prior buy level.

---

## Telegram Alert Trigger Rules (v3 NEW)

The Python pipeline (see `telegram_alert.py`) sends Telegram messages based on these rules:

### Alert Tier 1 — Action Required (sent immediately)

Send **ANY day** when ANY of the 3 benchmark scores ≥ 6.

Message format:
```
🚦 MF ENTRY SIGNAL — <DATE>

📊 Scores
Nifty 500: X/10 → <ACTION>
Midcap 150: Y/10 → <ACTION>
Smallcap 250: Z/10 → <ACTION>

💼 Funds to Deploy
<list only the funds with deploy >0%, with deploy % and INR if reserve known>

🧠 Why
<2 sentence reasoning citing the dominant dimensions>

⚠️ Blackout check
<if you bought in past 30 days on this cycle, flag it>
```

### Alert Tier 2 — Weekly Summary (every Sunday 9 AM IST)

Send a brief snapshot regardless of score, so the user knows the pipeline is alive.

Message format:
```
📅 Weekly Snapshot — <DATE>

Nifty 50: XX,XXX (DD -X.X%)
Midcap 150: XX,XXX (DD -X.X%)
Smallcap 250: XX,XXX (DD -X.X%)
VIX: XX.X

Current scores: N500=X, M150=Y, S250=Z
No deploy signal this week.

Pipeline: ✅ healthy
```

### Alert Tier 3 — Pipeline Error (only on failure)

If the script fails (data fetch error, score computation crash, Telegram API error):
```
⚠️ MF ALERT PIPELINE FAILURE
Date: <DATE>
Error: <error message>
Last successful run: <DATE>
```

### Anti-Spam Rules

- **Maximum 1 Tier-1 alert per day** even if running multiple times.
- **Suppress duplicate** alerts: if the score+action combination hasn't changed since
  yesterday's alert, send a brief "Signal unchanged from yesterday" line instead of
  the full message.
- **Blackout-active alerts** include a 🔒 prefix so you can scan-skip them.

### Optional: Strong Buy Multi-Channel

For score ≥ 9 (Strong Buy), the script can also call a phone number via Twilio (not
implemented by default — flag in code).
