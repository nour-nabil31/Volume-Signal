# 📊 VolumeSignal

A volume-based market conviction analyzer built in Python. Feed it any ticker and it computes five institutional volume indicators, combines them into a single **Conviction Score (0–100)**, and renders a 6-panel dashboard — all in one function call.

---

## The Core Idea

Price tells you what happened. Volume tells you how much conviction was behind it.

A stock that rises 4% on 3× normal volume is a fundamentally different signal than one that rises 4% on thin, quiet trading. The first has institutional money behind it. The second might be noise. VolumeSignal is built to tell the difference.

---

## The Conviction Score (0–100)

Five indicators each contribute a weighted number of points based on how precise and reliable they are as signals. The score measures how aligned the indicators are — not a prediction, but a measure of convergence.

| Indicator | Points | Why this weight |
|---|---|---|
| **Chaikin Money Flow (CMF)** | up to 25 | Highest weight — measures quality of buying pressure, not just quantity. Weights each day by where price closed within the high-low range. Most precise institutional flow indicator in the model. |
| **Price vs Rolling VWAP** | up to 20 | VWAP is the volume-weighted institutional cost basis. Price above VWAP = buyers paying above the institutional average = genuine demand. |
| **On-Balance Volume (OBV)** | up to 20 | Cumulative buying vs selling pressure. Captures longer-term flow that CMF's 20-day window can miss. OBV rising before price = institutional accumulation. |
| **Volume Activity Ratio** | up to 15 | MA20 / MA50 of volume. Is the market becoming more or less interested in this stock? Rising ratio = stock coming back onto the radar. |
| **A/D Line direction** | up to 10 | Accumulation/Distribution Line trend — same CLV logic as CMF but cumulative. |
| **Confirmation rate** | up to 10 | % of recent days where price direction and above-average volume agreed. |

### Conviction bands

| Score | Band | Meaning |
|---|---|---|
| 75 – 100 | **Strong Bullish** | All signals aligned. Volume strongly confirms the trend. |
| 50 – 74 | **Mild Bullish** | Mostly supportive. Monitor for continuation. |
| 35 – 49 | **Neutral** | Mixed signals. No clear volume conviction. |
| 20 – 34 | **Mild Bearish** | Volume mildly contradicts any bullish thesis. |
| 0 – 19 | **Strong Bearish** | Volume strongly contradicts the trend. |

---

## Features

- **6-panel dashboard** — Price vs VWAP, OBV, A/D Line, CMF, Volume bars, Volume Ratio
- **Warm finance visual theme** — cream, gold, burgundy palette distinct from standard charting libraries
- **HTML summary table** — all 10 indicators with signal badges rendered in Jupyter
- **Signal persistence filter** — only confirms a signal after it holds consistently for N consecutive days, eliminating single-day noise
- **Score history tracking** — automatically logs each day's score to a local JSON file, builds a running record per ticker
- **Dynamic** — works with any ticker, any time period, fully adjustable windows
- **EGX-aware** — calibrated for thin-liquidity markets like the Egyptian Exchange

---

## Example Output

### Summary table
Shows current readings for all 10 indicators with signal badges, conviction score progress bar, and overall verdict.

### Dashboard panels

| Panel | What it shows |
|---|---|
| Price vs VWAP | Price line vs rolling VWAP and long-run VWAP — green fill above, burgundy below |
| OBV | Cumulative buying pressure vs signal line — crossovers indicate momentum shifts |
| A/D Line | Accumulation/distribution trend — divergence from price is the key signal |
| CMF | Bar chart oscillating ±1 — green above zero, burgundy below, ±0.15 thresholds marked |
| Volume bars | Individual bars colored by direction with MA20 and MA50 overlaid |
| Volume Ratio | MA20/MA50 ratio — rising out of below-0.85 zone is one of the strongest signals |

---

## Installation

```bash
pip install numpy pandas yfinance matplotlib
```

---

## Usage

```python
from VolumeSignal import volume_signal, check_signal_consistency, get_score_history

# Minimal call — all defaults
volume_signal("ABUK.CA")

# US stock
volume_signal("AAPL")

# Custom windows
volume_signal("COMI.CA", cmf_window=14, vol_short=10, vol_long=30)

# Shorter history
volume_signal("ABUK.CA", start="2020-01-01")

# Stricter signal filter — 5 consecutive days instead of 3
volume_signal("ABUK.CA", required_days=5)

# Raise the buy threshold
volume_signal("ABUK.CA", threshold_buy=60)

# Table only — no plot
volume_signal("ABUK.CA", plot=False)

# Check consistency without running the full model
check_signal_consistency("ABUK.CA")

# View full score history for a ticker
get_score_history("ABUK.CA")
```

### Parameters

| Parameter | Default | Description |
|---|---|---|
| `ticker` | **required** | Any yfinance ticker — e.g. `"AAPL"`, `"ABUK.CA"` |
| `start` | `"2016-01-01"` | Historical data start date |
| `obv_window` | `20` | OBV signal line smoothing window |
| `cmf_window` | `20` | Chaikin Money Flow rolling window |
| `vol_short` | `20` | Short volume MA window |
| `vol_long` | `50` | Long volume MA window |
| `vwap_window` | `20` | Rolling VWAP window |
| `required_days` | `3` | Consecutive days score must hold before signal is confirmed |
| `threshold_buy` | `50` | Score must stay above this for a Buy confirmation |
| `threshold_sell` | `35` | Score must stay below this for a Sell confirmation |
| `history_file` | `"score_history.json"` | Path to persistent score history file |
| `plot` | `True` | Show the 6-panel dashboard |

---

## The Math

**Close Location Value (CLV)**
```
CLV = [(Close − Low) − (High − Close)] / (High − Low)
```
Ranges from +1 (closed at the high) to −1 (closed at the low). The foundation of both the A/D Line and CMF.

**Chaikin Money Flow**
```
MFV = CLV × Volume
CMF = Sum(MFV, window) / Sum(Volume, window)
```

**On-Balance Volume**
```
OBV(t) = OBV(t-1) + Volume   if Close > Close(t-1)
OBV(t) = OBV(t-1) − Volume   if Close < Close(t-1)
```

**Rolling VWAP**
```
VWAP = Sum(Close × Volume, window) / Sum(Volume, window)
```

**Volume Ratio**
```
Volume Ratio = MA(Volume, 20d) / MA(Volume, 50d)
```

---

## Best Horizon

The model is calibrated for a **2–3 week holding horizon**. The two heaviest-weighted indicators (CMF and VWAP) both use 20-day windows — signals that emerge from a 20-day window typically take 2–3 weeks to play out in price.

For shorter horizons reduce all windows. For longer horizons increase them:

```python
# 1-week signals
volume_signal("ABUK.CA", cmf_window=5, obv_window=5, vol_short=5, vol_long=20, vwap_window=5)

# 3-month signals
volume_signal("ABUK.CA", cmf_window=60, obv_window=60, vol_short=50, vol_long=200, vwap_window=60)
```

---

## Exit Checklist

Exit a position when **two or more** of the following trigger simultaneously:

- Conviction score drops below 35
- CMF crosses below zero
- Price closes below rolling VWAP for 3+ consecutive days
- OBV crosses below its signal line
- Volume ratio drops below 0.85

One trigger alone is noise. Two or more is a genuine signal.

---

## Signal Persistence Filter

A single day's score is often noise. The persistence filter requires a score to hold consistently above or below a threshold for N consecutive days before confirming a signal — eliminating false positives from one-off volume spikes or event-driven moves.

Every `volume_signal()` call automatically logs today's score to `score_history.json` and runs the consistency check. The result appears at the bottom of the summary table.

```python
# How it works in practice
# Day 1: score = 35 → logged. Not enough history yet.
# Day 2: score = 57 → logged. Two days, inconsistent.
#         Output: ⚠ INCONSISTENT — scores: [35, 57] — wait for consistency
# Day 3: score = 55 → logged. Still inconsistent (Day 1 was 35).
#         Output: ⚠ INCONSISTENT
# Day 4: score = 60 → logged. Last 3 days: [57, 55, 60]. All above 50.
#         Output: ✓ BUY CONFIRMED (3/3 days above 50)
```

Use the helper functions independently:

```python
# Check signal status without running the full model
check_signal_consistency("ABUK.CA")

# View the full score history for any ticker
get_score_history("ABUK.CA")
# Output:
# ───────────────────────────────────────────────────────
#   Score history — ABUK.CA  (4 days recorded)
# ───────────────────────────────────────────────────────
#   2026-05-28  |   35/100  ███████              NEUTRAL
#   2026-06-02  |   57/100  ███████████          MILD BULLISH
#   2026-06-03  |   55/100  ███████████          MILD BULLISH
#   2026-06-04  |   60/100  ████████████         MILD BULLISH
# ───────────────────────────────────────────────────────
```

Adjust strictness with parameters:

```python
volume_signal("ABUK.CA", required_days=3)    # default — faster signals, good for EGX
volume_signal("AAPL",    required_days=5)    # stricter — better for liquid US stocks
volume_signal("ABUK.CA", threshold_buy=60)   # raise the entry bar
volume_signal("ABUK.CA", threshold_sell=25)  # lower the exit bar
```

> **Design note:** the filter uses a rolling window of the last N days, not N days from the beginning. Recent readings matter more than older ones — a score history of [35, 57, 55, 60] confirms Buy on Day 4 because the last 3 days are all above 50, regardless of Day 1.

---

## Live Backtest — In Progress

> **Results will be published here after June 18, 2026.**

**Methodology:** 15 EGX stocks. Conviction scores recorded on May 28, 2026. Daily closing prices tracked through June 18, 2026. Accuracy measured overall and broken down by conviction band.

| Ticker | Entry Price | Conviction Score | Band | Decision |
|---|---|---|---|---|
| ELWA.CA | 2.03 | 85 | Strong Bullish | Buy |
| SPMD.CA | 0.41 | 75 | Strong Bullish | Buy |
| ETEL.CA | 97.63 | 72 | Mild Bullish | Buy |
| ACAMD.CA | 2.19 | 63 | Mild Bullish | Buy |
| MAAL.CA | 5.16 | 60 | Mild Bullish | Buy |
| JUFO.CA | 28.09 | 60 | Mild Bullish | Buy |
| GBCO.CA | 26.60 | 40 | Neutral | Hold |
| ARAB.CA | 0.20 | 30 | Mild Bearish | Sell |
| CCAP.CA | 5.12 | 28 | Mild Bearish | Sell |
| ARVA.CA | 8.08 | 27 | Mild Bearish | Sell |
| AIFI.CA | 1.76 | 27 | Mild Bearish | Sell |
| UEGC.CA | 1.32 | 10 | Strong Bearish | Sell |
| AJWA.CA | 132.92 | 10 | Strong Bearish | Sell |
| AMES.CA | 52.30 | 10 | Strong Bearish | Sell |
| ELEC.CA | 2.11 | 8 | Strong Bearish | Sell |

---

## Honest Limitations

Volume analysis assumes that volume meaningfully reflects institutional order flow — which holds better in liquid markets than thin ones. On EGX, single large trades can disproportionately move OBV and CMF without representing genuine market-wide conviction. The model is EGX-calibrated (using each stock's own volume history as the baseline) but this limitation is worth keeping in mind.

VolumeSignal is a conviction measurement tool, not a prediction engine. High conviction score + high volume ratio means the current environment is favorable. It does not guarantee price appreciation.

---

## Tech Stack

`Python` · `pandas` · `NumPy` · `yfinance` · `matplotlib`
