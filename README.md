# Pfizer (PFE) Stock Trading Strategy

This repository contains a machine learning-based single asset trading strategy for Pfizer Inc. (PFE), designed to make daily long/short position decisions.  
The strategy aims to outperform a buy-and-hold baseline using robust market features, an ensemble of models, and volatility-adjusted signal thresholds.

---

## Strategy Period

- January 1, 2010 — April 16, 2025
- Investment Amount: $1,000,000 USD per position  
- Position Type: Long/Short daily trading

---

## Data Extraction & Processing

- Historical data fetched via `yfinance`:
  - Assets: PFE, SPY, QQQ, XLV, JNJ, MRK, VIX, TNX (10-Year Treasury), DXY (US Dollar Index)
- Returns capped at 1st and 99th percentiles to limit outlier effects.
- Failsafe dummy data generation and robust error handling.

---

## Feature Engineering

60+ features grouped into 6 categories:

1. Basic Price & Volume  
   - Multi-timeframe returns (1, 2, 3, 5, 10, 20 days) with outlier clipping.
   - Rolling z-scores (price, volume) over 5, 10, 20, 50 days.
   - Overnight gaps, significant gap detection.

2. Technical Indicators  
   - RSI, MACD, Bollinger Bands, Stochastics, Williams %R, ROC, CCI.
   - All normalized; simple multiplicative interactions (e.g., RSI * VIX).

3. Volatility  
   - Rolling volatilities over 5, 10, 20, 60 days, scaled annually.

4. Market Context  
   - Bull/bear market regime (SPY 200-day MA).
   - VIX spikes, high/low volatility periods.
   - Rising rate detection, interest rate levels.

5. Sector & Peer  
   - XLV vs SPY relative strength.
   - Relative strength vs JNJ, MRK (10-day).

6. Pharmaceutical Specific  
   - Earnings month, year-end effects.
   - Pre/post COVID, vaccine phases.
   - Unusual volume spikes, overnight gaps.

Non-numeric logic handled as boolean features (e.g., is SPY above 200-day MA?). Converted to binary 0/1.

---

## Data Splits

- Train: 2010–2018  
- Validation: 2019–2020  
- Test: 2021–2025/04/16

---

## Modelling

- Ensemble: XGBoost, LightGBM, Random Forest, ElasticNet  
  - XGBoost: non-linear patterns, regularization.
  - LightGBM: efficient boosting.
  - RF: bias-variance diversity.
- Ensemble weights set by model correlation on validation.
- Models with negative/zero correlation are excluded.

## Target Variable
next_day_return_net = pct_change(close).shift(-1) – 0.001
0.1% Transactional cost is deducted

## Signal Generation and Position Handling

- Ensemble predicts next-day returns.
- Long/Short signal thresholds dynamically adjust using PFE’s recent volatility.
- Signal strength modulates $1M position by 0.5–2.0 based on prediction confidence.

## Backtesting and Evaluation

- PnL was calculated daily as Signal * Signal_Strength * next_day_return_net_already_cost_adjusted
- Metrics included total/annual return, volatility, Sharpe ratio, max drawdown, win rate, and number of trades
- Performance was compared against a PFE buy-and-hold baseline
