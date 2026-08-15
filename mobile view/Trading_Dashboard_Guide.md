# 📘 Institutional SMC Pro — Dashboard Terminology & Reference Guide

> Matches `Institutional_SMC_Pro.pine` as of 2026-08-14. This file is documentation only — it does not
> affect the indicator's behavior. If you edit the script's dashboard rows later, update this file too;
> it will otherwise silently go stale, which is exactly what happened to the previous version of this guide.

---

## 1. What This Dashboard Is

A single on-chart table (top-right by default) that condenses the script's trend/structure engine, an
adaptive multi-horizon price forecast, a trade-eligibility/risk engine, and a self-validation layer into
one HUD. It runs live only (`barstate.islast` — the current/forming bar), so historical bars never show it.

Two size modes:
- **Compact** (`Expand Dashboard` off, or Beginner display mode): 6 rows — Title, Trend, Timing, Score,
  Risk, Next-Day.
- **Expanded** (default): the 6 compact rows plus Validation, Tail-Risk, optionally Session Alignment,
  Opening Range, Next-Timeframe range, the five session-outlook rows (Pre/Reg/Post/Ovn/NxtDay), current
  session status, and the +30m→+6H hourly forecast ladder.

---

## 2. Glossary — Terms Used Across Multiple Rows

| Term | Meaning |
| :--- | :--- |
| **Conf** | Signal confidence — how strong the model's *own* directional signal is relative to that session's noise threshold (a signal-to-noise-ratio logistic transform), auto-adjusted by realized volatility clustering, VIX regime, and how far out the horizon is. **Not** an empirical win probability — a 90% Conf means "the model is very sure of its own bias," not "90% of similar setups closed correctly." |
| **CalP** | Calibrated probability — the actual directional win probability, built from a volatility-scaled tanh transform of expected return, blended with an online-fitted logistic (Platt) calibration once enough real outcomes exist. This *is* the empirical-style probability; Conf and CalP deliberately answer different questions and are shown side by side so they're never confused for one another. |
| **Emp:X%, N=Y** | Shown next to some probabilities (Pre/Reg/Post/Ovn/NxtDay rows). Emp% = how much of that probability is backed by the fitted Platt calibration vs. still the raw tanh-transform prior (0% = brand-new model, rises toward 100% as N grows). N = the number of resolved out-of-sample forecast-vs-outcome pairs the calibration fit has seen. Below ~40 resolutions, treat the probability as mostly the prior, not a trained estimate. |
| **R-multiple / R** | A trade's profit or loss measured in units of its own initial risk (the $ distance from entry to stop). `+2R` = the trade made twice what was risked; `-1R` = a full stop-out. Used instead of raw $ so trades of different sizes/instruments are comparable. |
| **ExpR** | Expected R — the mean realized R-multiple of past trades that opened at a given letter grade (A+/A/B/C). Shown as `ExpR:+0.42±0.15` once N≥5 — the `±` is one standard error, a rough gauge of how much sampling noise that mean carries. A grade can have a high win rate and still be unprofitable (small frequent wins, rare large losses) or the reverse, so ExpR matters alongside Win%, not instead of it. |
| **Brier score** | A probability-calibration score: mean squared error between stated probability and actual binary outcome. 0.25 = coin-flip baseline (no skill); lower = better calibrated; 0 = perfect. Two separate Brier scores are tracked — **Trade Brier** (entry-probability vs. trade outcome) and **NxtDay Brier** (next-day forecast vs. realized next close) — they measure different things and are never combined. |
| **DirAcc** | Directional accuracy — the rolling % of bars where the model's short-term bias correctly called the next bar's up/down direction. Bar-level, not trade-level. |
| **NAV Gap** | The gap between SOXL's actual cumulative return and a synthetic daily-reset 3x model's cumulative return computed from the same underlying-index moves. A direct, ongoing reality check on the leverage/decay/expense-drag model — not a trading signal itself. |
| **Grade (A+/A/B/C)** | A setup's percentile rank against its own rolling score history (90th/75th/60th/45th percentile cutoffs, recalculated continuously — not fixed thresholds). A+ ≈ top 10% of historical setups by composite score; C ≈ bottom of the range that still clears the minimum bar. Only A+/A/B setups are ever allowed to trigger a BUY/SELL signal. |
| **Heat** | Shown on the RISK row only while a position is open. The dollar amount (and % of account equity) that would actually be lost if the position were stopped out at its *current* stop level right now — a live "how much am I actually risking at this moment" figure, distinct from the R-multiple sizing used when the trade was opened. |
| **Alignment Score** | 0–100, centered at 50 (neutral). See §7 (Session Alignment) below. |
| **⚠️DIVERGENT** | Flag on the SCORE row: the long-side and short-side composite scores disagree by a wide margin from what the price forecast itself is saying — a sign the setup-quality engine and the forecast engine aren't currently telling the same story. |
| **⚠️WT-DEGEN** | Flag on the SCORE row: the adaptive factor weights (Trend/Liquidity/Momentum/Relative-Strength/Volatility) have all collapsed to their floor value, meaning the walk-forward learning loop currently trusts none of its own factors. Rare; worth investigating if it appears persistently. |

---

## 3. Row-by-Row Reference

| Row | Label | Shows | How to Read It |
| :--- | :--- | :--- | :--- |
| Title | `⚡SMC` | Ticker • chart timeframe | Sanity check the indicator is running on the chart you think it is. |
| Trend | `🚦TREND` | `Bullish 🟢` / `Bearish 🔴` / `Neutral (Correction) 🟡` | The script's own structural trend state (BOS/CHoCH + HTF alignment + momentum), not a raw moving-average cross. "Correction" means the trend flag is still set but price is currently pulling back against it. |
| Timing | `⏱TIMING` | The live signal verdict | See §6 for the exact message list — this is the single most important row, since it's built from the *same* booleans that gate real entries, never a separately-derived opinion. |
| Score | `📊SCORE` | `NN.N/100 Status [Grade Win%, ExpR:±SE, N=count]` | Status = Avoid/Watch/Tradable/Conviction, see §5. The bracketed footnote is the historical track record of trades opened at *this exact letter grade* — see Glossary for ExpR. |
| Risk | `⚠️RISK` | `Low/Medium/High (Vol:x.xx x \| RMSE:x.xx x)` + Heat if a position is open | Vol/RMSE ratios describe how volatile/unpredictable current conditions are relative to normal, not account risk. Heat (see Glossary) is the actual capital-at-risk figure. |
| Next-Day | `🔮NXT-DAY` | `Date BIAS (Exp Ret: ±X.XX% \| Up:XX%/Dn:XX%)` | Compact version of the detailed NxtDay row further down (expanded mode only) — same underlying number, shown here so it's visible even in compact mode. |
| Validation | `🧪VALID` | Trade Brier, directional accuracy, next-day hit rate, NAV Gap | The model grading its own track record — see Glossary for Brier/DirAcc/NAV Gap. `⚠LowN` means the backing sample (<30 resolved next-day forecasts) is still too small to trust the number shown. |
| Tail-Risk | `☢TAIL` | `P(>+2%):XX% \| P(<-2%):XX%` | Probability of a move *beyond* ±2% by next day — SOXL is 3x leveraged, so ±2% days are routine, not rare; this row exists so a "68% up" headline number isn't mistaken for "68% chance of a big move." |
| Session Alignment | `🎯ALIGN` | Decision + score + Current/Next-Regular + Pre/Post/Ovn direction glyphs | **Off by default** — only appears if "Use Session Alignment Confirmation" is enabled in Settings. See §7. |
| Opening Range | `ORB15m` / `ORB30m` | Realized or projected opening-range high/low | 🟢 = ORB window complete (levels final); 🟡 = still forming; 🔮 = projected (before market open). |
| Next-Timeframe | `Nxt<TF>` | Expected range for the *next single bar* of your chart's own timeframe | Short-horizon scalping range, not a session forecast. |
| Session outlook | `Pre` / `Reg` / `Post` / `Ovn` / `NxtDay` | Date, target price, expected range, % return, probability | See §4 for exactly what each is measured against — this is the part that most needs the base-price section below to interpret correctly. |
| Session status | `SESS` | Current session (color/icon-coded) + what's next | 🟡Pre / 🟢Market / 🔵Post / 🌙Ovn / 🔴Closed. |
| Hourly ladder | `+30m` … `+6H` | `$price (±%) [ConfXX% CalPXX%↑] $lo-hi` | Chained sequential forecast (+45m builds on +30m's estimate, etc.) — a coherent scenario path, not 8 independent forecasts. See Glossary for Conf vs. CalP. |

---

## 4. Base-Price Reference — What Every % Return Is Actually Measured Against

Every forecast row is a `(target price, base price)` pair, and the displayed `%` is always
`(target − base) / base × 100`. **Never compare two rows' raw dollar targets directly** — they can use
different bases, so only the `%` / probability figures are directly comparable across rows.

There are exactly two base-price conventions in this script:

- **Type A — "from here, where does the next occurrence of this session finish?"**
  Used by **Pre, Post, Overnight**. Base = the **current live price**, whatever session is active right
  now. Same measurement as "SOXL is up 2% today" — always relative to *now*, never to another session's
  close.

- **Type B — "where will the Regular session close relative to the last completed Regular close?"**
  Used by **Reg** (and by Next-Day's forecast internally). Base = **yesterday's Regular-session close**,
  even while today's Regular session is still open — today's own close isn't known yet, so the last
  genuinely completed reference point is still yesterday's.

| Row | Target | Base |
| :--- | :--- | :--- |
| Pre | Next Pre-Market close | Current live price |
| Reg | Today's (or next) Regular close | Yesterday's completed Regular close |
| Post | Next Post-Market close | Current live price |
| Ovn | Next Overnight close | Current live price — **except** while you're currently *inside* the Overnight session itself, where the base switches to **yesterday's completed Regular close instead** — the same base the Reg row uses (the live overnight tick is too illiquid/unrepresentative to anchor off; the code's own comment calls this "current regular market price," which is loose phrasing for the same prevDayC value, not a live price) |
| NxtDay (dashboard row/badge) | Next Regular session's close (one full cycle out) | Current live price |

**One intentional exception worth knowing about:** the visible NxtDay row/badge measures tomorrow's close
against the *current live price* (Type A — "how far is tomorrow's close from right now"), while the
internal Session Alignment engine (§7) measures that same tomorrow's-close target against *yesterday's*
Regular close instead (Type B — so it lines up on the same axis as today's Reg forecast, for the
divergence check). Two different numbers, both derived from the same underlying price forecast, answering
two different questions on purpose — not a bug.

### 4a. How "Increase" vs "Decrease" Is Actually Decided (the Noise Floor)

Computing `target − base` is only half the story. A forecast that's $0.01 above its base isn't a real
bullish call — it's float-math noise — so no row ever colors/arrows a direction purely because the raw
difference is positive or negative. Every row first checks whether the gap clears a **noise floor** around
the base price; below that floor, the row shows flat/neutral (`--`) instead of a false-confidence arrow.
Concretely: `target ≥ base + floor` → 📈 up; `target ≤ base − floor` → 📉 down; otherwise → `--` neutral.

**The floor size is not the same everywhere** — this matters because the same $ move can read as "up" on
one row and "flat" on another:

| Row(s) | Noise Floor | Units |
| :--- | :--- | :--- |
| Pre / Reg / Post / Ovn (session outlook rows) | `ATR × 0.05` | Dollars, added/subtracted from the base price directly |
| +30m → +6H hourly ladder | `max(instrument min tick, ATR × 0.10)` | Dollars — **twice** as strict as the session-outlook rows, since a single-bar-scale forecast needs a clearer signal before committing to a direction |
| NxtDay (dashboard badge + detailed row) | Fixed `±0.10%` | Percent of the expected return itself — the **one** row that does **not** scale with ATR |
| Session Alignment engine (Current-Reg/Next-Reg/Pre/Post/Ovn legs, §7) | `ATR × 0.10`, expressed as a % of price | Percent — same underlying sensitivity as the hourly ladder, used internally for the LONG BIAS/SHORT BIAS/WAIT decision rather than for an on-screen icon |

**Practical effect:** on a quiet, low-ATR day, a real $0.30 expected move might not clear the Reg row's
`ATR × 0.05` floor if ATR itself is small that day — the row shows `--` rather than a direction. On a
volatile, high-ATR day, that same $0.30 move clears the floor easily. The bar moves with the instrument's
*own* recent volatility — except NxtDay, which always uses the same flat 0.10% regardless of how volatile
SOXL has actually been lately.

### 4b. Worked Example — One Scenario, Every Row

Same moment in time throughout: SOXL is inside the **Regular Market** session, ATR = **$3.00**, yesterday's
completed Regular close (`prevDayC`) = **$140.00**, and the current live price right now = **$143.50**.

| Row | Target (the model's forecast) | Base | Calculation | Result |
| :--- | :--- | :--- | :--- | :--- |
| **Reg** | $144.20 (today's expected close) | $140.00 *(yesterday's close — Type B)* | `(144.20 − 140.00) / 140.00 × 100` | **+3.00%**, floor = `3.00 × 0.05` = $0.15 → $144.20 clears $140.15 → **📈 UP** |
| **Pre** | $144.80 (next Pre-Market forecast) | $143.50 *(live price right now — Type A)* | `(144.80 − 143.50) / 143.50 × 100` | **+0.91%**, floor = $0.15 → $144.80 clears $143.65 → **📈 UP** |
| **Post** | $143.20 (next Post-Market forecast) | $143.50 *(live price — Type A)* | `(143.20 − 143.50) / 143.50 × 100` | **−0.21%**, floor = $0.15 → $143.20 clears $143.35 (the down side) → **📉 DOWN** |
| **Ovn** *(normal — not currently inside Overnight)* | $143.55 (next Overnight forecast) | $143.50 *(live price — Type A)* | `(143.55 − 143.50) / 143.50 × 100` | **+0.03%** — the raw $0.05 gap is *smaller* than the $0.15 floor → neither side clears → **`--` flat/neutral**, even though the raw forecast is technically higher |
| **Ovn** *(special case — you're currently inside Overnight)* | $142.50 (next Overnight forecast) | $140.00 *(yesterday's Regular close — switches to Type B, see §4)* | `(142.50 − 140.00) / 140.00 × 100` | **+1.79%**, floor = $0.15 → clears easily → **📈 UP** |
| **NxtDay** | $146.00 (next Regular close, one cycle out) | $143.50 *(live price — Type A)* | `(146.00 − 143.50) / 143.50 × 100` | **+1.74%**, floor = fixed `0.10%` (not ATR-scaled) → clears easily → **📈 UP** |

Two things this example is built to show:
1. **Pre/Post/Ovn all use the same live $143.50 base** (Type A) *except* Ovn switches to the $140.00
   Regular-close base the moment you're physically inside the Overnight session (Type B) — same target
   price convention, different base depending on where "now" sits.
2. **The Ovn "normal" row is the one that goes flat** — a real, positive $0.05 forecast still displays as
   `--` because it doesn't clear that row's $0.15 noise floor. This is the noise floor doing its job: a
   $0.05 edge on a $3.00-ATR instrument isn't a real directional call, so the dashboard doesn't pretend it
   is.

---

## 5. Probability & Confidence — How They're Actually Computed

**CalP (calibrated probability):**
1. Expected return is converted to a z-score against an ATR-scaled volatility measure, then passed through
   a hyperbolic tangent (bounded, symmetric transform).
2. That raw estimate is blended with an online-fitted logistic (Platt) calibration — two coefficients
   updated by gradient descent every time a real forecast resolves (both a ~1-bar-ahead check and a
   once-per-session next-day check feed this).
3. Early on (before enough real resolutions exist), CalP is almost entirely the raw tanh estimate; it
   shifts toward the fitted calibration as more history accumulates (see Emp/N in the Glossary).

**Conf (signal confidence), hourly ladder + session rows only:**
1. Signal-to-noise ratio = |bias| ÷ that session's own noise threshold (Overnight needs the strongest
   signal to be "actionable," Regular Market the least — thin-liquidity sessions get a higher bar).
2. Passed through a logistic curve whose steepness self-calibrates against a blend of the short-horizon
   and next-day Brier scores (better real-world calibration → steeper curve → more decisive Conf values).
3. Scaled down for horizon decay (further out = less confident, all else equal) and elevated VIX.

Neither formula uses a fixed per-session multiplier table — both continuously adapt to realized volatility,
calibration accuracy, and how far out the forecast reaches.

---

## 6. Score Status Thresholds

`activeMinScore` below is **not** a fixed number — it's the model's own dynamic 60th-percentile score
threshold, recalculated continuously from its own rolling history.

| Composite Score | Status | Meaning |
| :--- | :--- | :--- |
| < 40 | `Avoid 🔴` | Below the minimum bar regardless of anything else. |
| 40 – just under `activeMinScore` | `Watch 🟡` | Developing, not yet at the model's own trade-eligibility bar. |
| `activeMinScore` – `activeMinScore+10` (min 80) | `Tradable 🟢` | Clears the eligibility bar. |
| Above that | `Conviction 🔥` | High-confluence setup, well above the model's own typical bar. |

---

## 7. Timing Row — Every Possible Message

The Timing row always reads the *exact same* booleans that gate real entries — it can never say "BUY"
while a real signal fails to fire, or vice versa.

**While flat (no open position):**
| Message | Cause |
| :--- | :--- |
| `🟢 BUY` (+ `(approx R:R)` if the reward target is synthetic, not a real structural level) | Long entry conditions all satisfied. |
| `🔴 SELL SHORT` (+ `(approx R:R)`) | Short entry conditions all satisfied. |
| `Suspended (Edge Neg)` | Position sizing came out to zero shares — no trade regardless of signal. |
| `Suspended (Shorting Disabled)` | Trend is bearish, everything else would pass, but shorting is turned off for this instrument/settings. |
| `Suspended (R:R < X.Xx)` | Reward-to-risk ratio below the required minimum (2.0x normally, 3.0x if the target had to fall back to a synthetic ATR-multiple level). |
| `Suspended (Await Pullback)` | Price is judged overextended in the trade's own direction. |
| `Suspended (Outside Window)` | Outside a kill zone, or relative volume too thin. |
| `Suspended (Score/Grade Below Min)` | Composite score or letter grade hasn't cleared the bar. |
| `Suspended (Execution Filters)` | All of the above passed, but a lower-level execution-quality gate (ATR shock, excessive gap, thin liquidity, doji-shock bar, daily loss limit hit, macro shock, VIX extreme, or an active loss-streak cooldown) is blocking entry. |
| `Suspended (Rangebound)` | No directional trend at all right now. |

**While in a position:**
| Message | Cause |
| :--- | :--- |
| `🟢 HOLD LONG` / `🔴 HOLD SHORT` | Position open, stop distance still healthy. |
| `🚨 Near SL (Long/Short)` | Price has closed to within 0.5×ATR of the stop — no action implied, just a proximity warning. |

---

## 8. Session Alignment (`🎯ALIGN`) — Optional Confirmation Layer

**Off by default.** Enable via Settings → "🎯 Session Alignment" → "Use Session Alignment Confirmation."
When off, this row is hidden and has zero effect on signals.

**What it does when enabled:** requires the **Current-Regular** and **Next-Regular** forecasts (both
measured Type-B, vs. yesterday's close — see §4) to agree on direction, beyond an ATR-scaled noise floor,
before a BUY/SELL is allowed to fire — layered on top of, not replacing, the full existing eligibility
engine (trend/score/R:R/timing/execution quality all still apply in full).

| Decision | Meaning |
| :--- | :--- |
| `LONG BIAS` / `SHORT BIAS` | Current-Reg and Next-Reg both agree, and at least 2 of the 3 secondary legs (Pre/Post/Ovn) agree too. |
| `LONG BIAS (Low Conv)` / `SHORT BIAS (Low Conv)` | Current-Reg and Next-Reg agree, but the secondary legs are mixed or opposed — still confirms the gate, just flagged as weaker. |
| `WAIT (Reg Divergence)` | Current-Reg and Next-Reg point in **opposite** directions — blocks both BUY and SELL regardless of what the rest of the engine says. |
| `NEUTRAL/MIXED` | Neither a clear agreement nor a clear conflict (e.g. one leg flat/near-zero). |

The **Alignment Score** (0–100, centered at 50) is a confidence-weighted blend of all five legs
(Current-Reg 40%, Next-Reg 30%, Pre/Post/Ovn 10% each) — display-only, not itself a gate. Those weights
are a documented starting heuristic, not a learned/validated allocation.

---

## 9. Mobile vs. Desktop

Enabling "Mobile Friendly View" shows the *same* rows and the *same* full-detail text as desktop —
nothing is hidden or shortened. Only the layout changes: every cell is narrowed and left-aligned instead
of desktop's auto-width, centered columns, since TradingView's mobile app clips overflowing cell text
instead of wrapping it, so long strings (Validation, Score footnote, session-outlook rows) may wrap onto
more visual lines on a phone than on a wide desktop monitor — the content itself is identical.

---

## 10. Quick Cheat Sheet

| See This | It Means | Do This |
| :--- | :--- | :--- |
| `TREND: Bearish 🔴` | Structure is down. | Don't go long; look only at shorts (if enabled) or sit out. |
| `TIMING: 🟢 BUY` | Every gate passed. | This is the model's actual entry signal. |
| `TIMING: Suspended (...)` | Something specific is blocking entry. | Read §7 to see exactly what and why. |
| `SCORE: Avoid 🔴` | Setup quality below the floor. | Don't trade regardless of what else looks good. |
| `RISK: ... Heat:$X (Y%)` | You're in a trade; this is your live capital at risk right now. | Compare against your own risk tolerance, not just the original R sizing. |
| `NXT-DAY: BEARISH ↓` | Tomorrow's forecast leans down, measured from right now. | Context for planning, not a standalone entry trigger. |
| `🎯ALIGN: WAIT (Reg Divergence)` | Today's and tomorrow's Regular forecasts disagree. | If this gate is enabled, new entries are paused until they agree. |
