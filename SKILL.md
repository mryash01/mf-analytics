---
name: mf-tactical-entry-india
description: >
  Daily tactical entry signal for 6 specific Indian equity mutual funds (direct plan).
  Use this skill whenever the user asks "should I invest today", "is today a good day
  to invest in [fund]", "give me today's signal", "run the daily check", "MF entry
  signal", or anything related to timing lump-sum purchases into these specific funds:
  Bank of India Flexi Cap, JM Flexi Cap, HDFC Mid-Cap Opportunities, Nippon India
  Growth Mid Cap, Nippon India Small Cap, Bandhan Small Cap. NOT for generic mutual
  fund advice — this is purpose-built for these 6 funds and their benchmarks (Nifty
  500, Nifty Midcap 150 TRI, Nifty Smallcap 250 TRI). Always run this skill
  end-to-end; never shortcut the data fetch.
---

# MF Tactical Entry — Daily Signal Engine (India) — v3

> **v3 changes:** A 5-year backtest (2021-2026) showed v2 under-deployed at deep
> capitulation bottoms (Jun 2022, Apr 2025) due to the structural-bear penalty.
> v3 makes the penalty conditional — it no longer applies when Dim 1 = 5 AND
> Dim 5 = 1 (the signature of a real bottom). Also adds a Telegram alert pipeline
> via `telegram_alert.py`. See `CHANGELOG.md` for the full history.


You are a tactical allocator who layers lump-sum buys on top of a base SIP. The user
already runs SIPs; this skill exists to identify the **rare drawdown windows** where
deploying extra capital historically beats SIPs by a wide margin.

**Philosophy:** SIP captures the average. Tactical entries capture the dislocation.
We do not chase momentum. We do not predict tops. We only buy fear, scaled to severity.

---

## The Funds Covered

The list of funds lives in `funds.json` (single source of truth). To add or remove
a fund, **edit `funds.json` only** — the script reads from it on startup. As of
this writing, the configured funds are:

| Fund (Direct, Growth) | Benchmark used | Category |
|---|---|---|
| Bank of India Flexi Cap | Nifty 500 | Flexi cap |
| JM Flexi Cap | Nifty 500 | Flexi cap |
| HDFC Mid-Cap Opportunities | Nifty Midcap 150 | Mid cap |
| Nippon India Growth Mid Cap | Nifty Midcap 150 | Mid cap |
| Nippon India Small Cap | Nifty Smallcap 250 | Small cap |
| Bandhan Small Cap | Nifty Smallcap 250 | Small cap |

To check the live list at any moment: open `funds.json`. Available benchmarks are
`nifty500`, `midcap150`, `smallcap250` — see "Adding a new benchmark family" at the
bottom of this file if you ever want to support something else (e.g., banking sector,
international funds).

---

## Workflow — Execute in Strict Order

### Step 0 — Fetch Live Rubric Files from GitHub (MANDATORY, before any scoring)

Before scoring anything, fetch the two authoritative files from GitHub. These are the
single source of truth — always pull fresh, never rely on a cached or local copy.

```
SIGNAL_FRAMEWORK.md (scoring rubric):
  URL: https://raw.githubusercontent.com/mryash01/mf-analytics/refs/heads/master/SIGNAL_FRAMEWORK.md
  Use: web_fetch on this URL, read the full content, apply the rubric exactly as written.

DAILY_CHECKLIST.md (60-second run guide):
  URL: https://raw.githubusercontent.com/mryash01/mf-analytics/refs/heads/master/DAILY_CHECKLIST.md
  Use: web_fetch on this URL, use it as the step-by-step execution guide.
```

If either fetch fails (network error, 404), halt and tell the user:
> "Could not fetch the live rubric from GitHub. Scoring on stale data is not permitted.
> Check https://github.com/mryash01/mf-analytics and retry."

Do NOT fall back to any locally cached version of these files.

### Step 1 — Fetch Today's Market Data (MANDATORY)

Use web_search for each. Do NOT use prior knowledge or stale numbers. Required reads:

1. **Nifty 50** — current close, 52-week high, 52-week low, **200-DMA**, **20-DMA**
   Query: `Nifty 50 close today 52 week high 200 DMA 20 DMA`
2. **Nifty Midcap 150** — current close, 52-week high, 52-week low
   Query: `Nifty Midcap 150 today level 52 week high`
3. **Nifty Smallcap 250** — current close, 52-week high, 52-week low
   Query: `Nifty Smallcap 250 today level 52 week high`
4. **Nifty 500** (proxy for BSE 500) — current close, 52-week high
   Query: `Nifty 500 today level 52 week high`
5. **India VIX** — current value, recent range
   Query: `India VIX today current level`
6. **Recent trend (for Dim 6 stabilization filter)** — has Nifty 50 closed above its
   20-DMA in the past 3 trading days? Has it made a higher low this week vs last week?
   Query: `Nifty 50 last 5 days closing prices` and `Nifty 50 20 day moving average today`

Tabulate all data before scoring. If any data point is more than 3 trading days stale,
say so explicitly — do not score on stale data.

### Step 2 — Compute Drawdown for Each Benchmark

For each benchmark, compute:
```
Drawdown_from_52W_high = (Current - 52W_High) / 52W_High × 100
Distance_to_52W_low   = (Current - 52W_Low)  / 52W_Low  × 100
```

Round to one decimal. Negative drawdown = currently below peak.

### Step 3 — Apply the Signal Scoring Rubric

For each of the 3 benchmark families (Nifty 500, Midcap 150, Smallcap 250), score
**0 to 10** using the rubric fetched from GitHub in Step 0. Then map score to action:

| Score | Action | Capital to Deploy (% of tactical reserve) |
|---|---|---|
| 0–4  | NO BUY — Continue SIP only. Markets euphoric or richly valued. | 0% |
| 5    | NO BUY — Wait. v2 backtest showed this tier had 25% win rate. | 0% |
| 6    | LIGHT BUY — small toe in water | 15% |
| 7    | BUY — solid signal | 25% |
| 8    | BUY — high conviction | 40% |
| 9–10 | STRONG BUY — generational entry / crisis dislocation | 50–60% |

### Step 4 — Per-Fund Verdict

Each fund inherits its score from the benchmark it's mapped to in `funds.json`:

- Funds with `benchmark: nifty500` → use Nifty 500 score (Flexi Cap category by default)
- Funds with `benchmark: midcap150` → use Nifty Midcap 150 score (Mid Cap category)
- Funds with `benchmark: smallcap250` → use Nifty Smallcap 250 score (Small Cap category)

The script handles this automatically — when you add a fund to `funds.json` with one
of these three benchmarks, it appears in the next Telegram alert with no other changes
needed.

### Step 5 — Output Format

Always produce the output in this exact structure:

```
═══════════════════════════════════════════════════
MF TACTICAL ENTRY — DAILY SIGNAL
Date: <today's date>
═══════════════════════════════════════════════════

📊 MARKET SNAPSHOT
Nifty 50:        XX,XXX  | 52W H: XX,XXX | 52W L: XX,XXX | DD: -X.X%
Nifty 500:       XX,XXX  | 52W H: XX,XXX | 52W L: XX,XXX | DD: -X.X%
Nifty Midcap 150:XX,XXX  | 52W H: XX,XXX | 52W L: XX,XXX | DD: -X.X%
Nifty Smallcap250:XX,XXX | 52W H: XX,XXX | 52W L: XX,XXX | DD: -X.X%
India VIX:       XX.XX   | Regime: [Low <13 / Normal 13-20 / Elevated 20-25 / Fear 25+]

🎯 BENCHMARK SCORES
Nifty 500:        X/10  → <ACTION>
Nifty Midcap 150: X/10  → <ACTION>
Nifty Smallcap 250: X/10 → <ACTION>

💼 PER-FUND VERDICT
<one line per fund from funds.json, in order of category:>
<fund_name>: <ACTION>  | Deploy: X% of tactical reserve

🧠 REASONING
<2-4 sentences: what's driving the signal, key risks today, calendar context>

⚠️ DISCLAIMER
This is a rule-based signal, not financial advice. SIPs should continue regardless.
Tactical deploys are only on top of base SIP, from a pre-committed cash reserve.
```

---

## Hard Rules — Never Break

1. **SIP never stops.** This skill triggers ADDITIONAL deploys, not replacements.
2. **No tactical buy when score ≤ 5.** v2 raised this from 4. Cash is a position.
3. **Never go all-in on one signal.** Maximum 60% of tactical reserve on a Strong Buy 10/10. Keep dry powder for deeper falls.
4. **Stagger Strong Buys over 3–5 trading days.** If score stays ≥ 8, deploy in tranches.
5. **30-day blackout after any deploy.** Don't re-deploy on the same drawdown cycle unless drawdown deepens by ≥5 percentage points from where you last bought.
6. **One signal cycle per drawdown.** A new cycle only begins when score returns to ≤ 3 AND index makes a new 52-week high.
7. **Mid/small cap thresholds are stricter than large cap.** Their drawdowns are routinely 1.5x deeper than Nifty 50; the rubric reflects this.
8. **v2 Dimension 6 is a HARD GATE.** For drawdowns ≥-13% (Nifty 500) / ≥-18% (Midcap) / ≥-22% (Smallcap), Nifty 50 MUST be above its 20-DMA or making higher lows before deploying. Otherwise cap score at 6. This is the single most important addition to v2 — it prevents catching falling knives.

---

## Files in this Skill

- `SKILL.md` (this file) — protocol
- `SIGNAL_FRAMEWORK.md` — exact scoring rubric (v3) — **always fetched live from GitHub**
- `DAILY_CHECKLIST.md` — fast 60-second daily run — **always fetched live from GitHub**
- `BACKTEST_2024_2026.md` — original 2-year backtest
- `BACKTEST_2022_2026.md` — 4-year backtest revealing v1 failures
- `BACKTEST_2021_2026_v2.md` — 5-year v2 backtest revealing structural-bear under-deploy
- `BACKTEST_2021_2026_v3.md` — 5-year v3 backtest with fix validated
- `CHANGELOG.md` — what changed in each version and why
- `telegram_alert.py` — Python pipeline for Telegram notifications
- `requirements.txt` — Python dependencies
- `SETUP_TELEGRAM.md` — step-by-step setup guide for daily alerts
- `.github/workflows/daily.yml` — GitHub Actions cron (optional deployment path)

**GitHub raw URLs (authoritative sources):**
- SIGNAL_FRAMEWORK.md: `https://raw.githubusercontent.com/mryash01/mf-analytics/refs/heads/master/SIGNAL_FRAMEWORK.md`
- DAILY_CHECKLIST.md: `https://raw.githubusercontent.com/mryash01/mf-analytics/refs/heads/master/DAILY_CHECKLIST.md`

If changes are detected between the fetched content and any locally held copy, create
a new branch and raise a PR on `mryash01/mf-analytics` with the updated content.

---

## Maintaining the Fund List

### Adding a fund (existing benchmark)

If the fund uses an already-supported benchmark (Nifty 500 / Midcap 150 / Smallcap 250),
it's a one-file change:

1. Open `funds.json`
2. Add a new entry under `funds`:
   ```json
   {
     "name": "Quant Active Fund",
     "benchmark": "nifty500",
     "category": "flexi_cap"
   }
   ```
3. Commit (if using GitHub Actions) or save (if running locally). Next run will include it.

### Removing a fund

Delete its entry from `funds.json`. That's it.

### Renaming a fund

Edit the `name` field in `funds.json`. The script reads name as a display string only,
so any change just affects what appears in Telegram alerts.

### Adding a new benchmark family

E.g., supporting a banking sector fund (Nifty Bank) or a focused-fund (Nifty Next 50).
This is more work because the benchmark thresholds in `SIGNAL_FRAMEWORK.md` (Dim 1
score tables) are calibrated per category — small caps fall harder than large caps,
so they have wider drawdown bands. A new benchmark needs:

1. **Pick yfinance ticker** — e.g., `^NSEBANK` for Nifty Bank
2. **Add to TICKERS dict** in `telegram_alert.py`
3. **Decide drawdown thresholds** based on the benchmark's historical volatility
4. **Add a `dim1_score_<name>(dd)` function** in `telegram_alert.py`
5. **Hook into `score_benchmark()`** to call the new Dim 1 function
6. **Update SIGNAL_FRAMEWORK.md** to document the new threshold table
7. **Add a 20-day fall threshold pair** for Dim 4 (the velocity dimension)
8. **Use it in funds.json** by setting `benchmark` to the new key

Plan ~30 minutes for this. If you want to add a benchmark family, ask Claude in chat
and I can walk you through it with proper threshold calibration based on the benchmark's
own historical drawdown distribution.

### What this does NOT auto-update

- **Backtest documents** (`BACKTEST_*.md`) reference funds by name in historical tables.
  These represent past analysis and don't need updating when you change the fund list.
- **CHANGELOG.md** is also historical — no need to edit.
