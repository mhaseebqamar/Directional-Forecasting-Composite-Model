# NVDA Multi-Horizon Directional Signal Model (Rule-Based)

A rule-based quantitative signal model that generates BUY / HOLD / SELL signals for NVIDIA (NVDA) and checks their directional accuracy over 9 forward horizons, from 1 week (5 trading days) to 6 months (126 trading days). It uses no machine learning: the signals come from hand-designed indicator blocks combined into a weighted composite score.

## Project context

This is my individual submission to a Quantitative Analyst case study assessment set by Devvnex, published here with their permission. The brief set strict constraints, which shaped the design:

* NumPy, Pandas and Matplotlib only (no TA-Lib, Backtrader, Zipline or SciPy)
* No regression, machine learning or neural networks
* No hypothesis testing or p-values
* Moving averages allowed only as a secondary filter, not as the main signal
* Look-ahead bias must be avoided
* Target: 70% directional accuracy, where accuracy = correct BUY and SELL signals ÷ all BUY and SELL signals (HOLD excluded)

## What this project does

* Loads 333 trading days of NVDA daily OHLCV data (2 Jan 2024 – 30 Apr 2025, Investing.com export), with a loader that handles the export format (BOM, US date format, reverse order, volume strings such as `411.25M`)
* Builds every indicator from scratch in NumPy/Pandas (rate of change, On-Balance Volume, Donchian channel, Wilder's True Range / ATR)
* Combines four signal blocks, each normalised with a rolling z-score and bounded with `tanh`, into one composite score:

| Block | Logic | Weight |
|---|---|---|
| Momentum | Rate of change over 5, 21 and 63 days | 40% |
| Trend position | Close position within the 20-day Donchian channel | 35% |
| Volume | 20-day OBV change, amplified by above-average volume | 15% |
| Mean reversion | Contrarian stretch from the 20-day median, in ATR units | 10% |

* Converts the score into BUY / HOLD / SELL only when four guardrails agree:
  1. **Regime gate**: price above a rising 50-day SMA for BUY, below a falling one for SELL
  2. **Momentum confirmation**: medium- or long-term momentum must agree with the signal
  3. **Extreme-stretch gate**: no BUY above +2 ATR stretch, no SELL below −2 ATR
  4. **Pullback-within-trend**: no BUY near the top of the Donchian range, no SELL near the bottom
* Uses asymmetric thresholds (BUY > +0.15, SELL < −0.25), so a SELL needs stronger evidence than a BUY
* Splits the 214 evaluable days (after indicator warm-up) chronologically: 70% train (24 Jun 2024 – 27 Jan 2025, 149 days) and 30% test (28 Jan – 30 Apr 2025, 65 days)
* Reports accuracy for every horizon next to an **always-BUY baseline** on the same period, so model skill can be told apart from simply riding the trend

## Results

The model is very selective: across the 214 evaluable days it gave 9 BUY, 26 SELL and 179 HOLD signals.

**Test period (28 Jan – 30 Apr 2025)**

| Horizon | Signals | Model accuracy | Always-BUY baseline | Edge (pp) |
|---|---|---|---|---|
| 1W | 20 SELL | 50.0% | 48.3% | +1.7 |
| 2W | 15 SELL | 53.3% | 41.8% | +11.5 |
| 3W | 15 SELL | 80.0% | 30.0% | +50.0 |
| 1M | 15 SELL | 86.7% | 9.1% | +77.6 |
| 2M | 7 SELL | 100.0% | 0.0% | +100.0 |
| 3M | 2 SELL | 100.0% | 0.0% | +100.0 |
| 4M – 6M | — | not evaluable | — | — |

**Summary by segment**

| Segment | Horizons with signals | Horizons ≥ 70% | Horizons beating baseline by ≥ 5 pp | Mean edge (pp) |
|---|---|---|---|---|
| Full period | 9 | 0 | 3 | −11.4 |
| Train | 8 | 1 | 0 | −35.1 |
| Test | 6 | 4 | 5 | +56.8 |

**How to read these numbers**

* **Every test-period signal was a SELL.** The test window covers NVDA's decline in early 2025, so the strong test results show the model correctly staying bearish through one falling market. They do not show that it works across different market conditions.
* **The signal counts are small, and the signals overlap.** The 100% results at 2M and 3M rest on 7 and 2 signals. Signals on consecutive days share most of their forward window, so they are not independent observations.
* **Results in the training period were weak.** Accuracy was 13–33% on most horizons, and none of the SELL signals in the training period was correct. Over the full period, no horizon reached 70%.
* **4M–6M could not be tested out of sample**, because the 65-day test window is shorter than those horizons.
* Together, these results point to a model whose performance depends heavily on the market regime. It would need testing over a longer history with several up and down phases before any conclusion about real predictive skill.

## Tech stack

* Python
* NumPy, pandas: indicators, signals, evaluation
* Matplotlib: charts

## Project structure

```
Directional-Forecasting-Composite-Model/
├── Nvidia_Case_study.ipynb    # Full notebook: data → indicator blocks → composite → signals → evaluation → charts
└── README.md
```

Running the notebook creates an `outputs/` folder with the accuracy tables (`accuracy_table.csv`, `final_accuracy_table.csv`) and three charts: price with signals, indicator breakdown, and accuracy vs baseline.

## How to run

1. Download NVDA daily price history for 2 Jan 2024 – 30 Apr 2025 from Investing.com as CSV.
2. Set `CSV_PATH` in the first code cell to the file location.
3. Run all cells from top to bottom. The model has no random elements, so the results are fully reproducible.

## Honest limitations

* **One short test window in a single market regime** (65 trading days, a falling market). The test results cannot be generalised.
* **Small, overlapping samples.** Accuracy at longer horizons rests on very few signals.
* **Weak results in the training period and over the full period**, as shown above.
* **Weights and thresholds were set by judgement, not fitted.** That avoids overfitting to the test data, but it also means the design choices have not been tested systematically.
* **No trading simulation.** The project measures directional accuracy only, not returns, transaction costs or risk.
* This is an analytical exercise, not a trading system, and should not be used for investment decisions.
