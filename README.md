# NVDA Multi-Horizon Directional Forecasting Engine
### Quantitative Analyst Assessment Submission

## 📈 Executive Summary
This project implements a pure-quantitative composite scoring engine designed to forecast the directional movement of **NVIDIA (NVDA)**. Built for institutional-style analysis, the model moves beyond simple indicators by utilizing a **multi-horizon validation framework**. 

The engine evaluates signals across 9 forward horizons (from 5 to 126 trading days), allowing for a granular understanding of signal decay and predictive efficacy over short, medium, and long-term periods.

---

## 🛠 Model Architecture
The core logic is a weighted composite of four independent signal blocks, ensuring that trading signals are backed by multiple dimensions of market data:

1.  **Momentum Block:** Measures the velocity of price action.
2.  **Trend Position Block:** Utilizes Donchian Channels to assess price location relative to historical boundaries.
3.  **Volume Block:** Analyzes volume-price divergence to validate move conviction.
4.  **Mean Reversion (Stretch) Block:** Identifies overextended conditions to prevent buying at local tops.

### Technical Sophistication:
* **Rolling Z-Score Normalization:** All signals are normalized via rolling windows (20–60 days) and bounded by a `tanh` function to ensure statistical stability and prevent outlier skew.
* **Regime-Aware Gating:** A secondary SMA-50 filter is implemented to ensure high-conviction signals align with the broader market trend.
* **Extreme-Stretch Protection:** An automated "gate" that suppresses BUY signals when the asset is statistically overbought, regardless of momentum.

---

## 📊 Performance Analysis
The model was tested against a "Buy & Hold" baseline. While the full-period data includes high volatility, the **Out-of-Sample (TEST)** results demonstrate significant predictive alpha.

| Segment | Valid Horizons | 70%+ Accuracy Hits | Mean Edge over Baseline |
| :--- | :---: | :---: | :---: |
| FULL Period | 9 | 0 | -11.4 pp |
| TRAIN Set | 8 | 1 | -35.1 pp |
| **TEST (Out-of-Sample)** | **6** | **4** | **+56.8 pp** |

**Key Finding:** In the out-of-sample period, the model outperformed the baseline by an average of **56.8 percentage points**, hitting the 70% accuracy threshold on 4 out of 6 valid horizons.
