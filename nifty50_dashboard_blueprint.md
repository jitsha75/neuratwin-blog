# Nifty50 Investment Dashboard Blueprint

This guide helps you build a dashboard that can **actively search and rank Nifty50 stocks** for investment.

## 1) Goal

Create a dashboard that:

1. Pulls live/recent market data for Nifty50 stocks.
2. Calculates technical + fundamental indicators.
3. Scores stocks using configurable weights.
4. Displays top candidates with rationale.
5. Lets you backtest and tune strategy.

> Important: This is for educational use, not financial advice.

---

## 2) Recommended Stack

- **Frontend:** Streamlit (fast to build) or Plotly Dash.
- **Backend data:** Python + pandas.
- **Market data:** NSE/BSE-compatible provider (or yfinance prototype).
- **Storage:** SQLite/PostgreSQL for historical snapshots.
- **Scheduling:** cron/APScheduler for periodic refresh.

---

## 3) Data Model

Create one universe table and one metrics snapshot table.

### `nifty50_universe`

- symbol
- company_name
- sector
- index_weight

### `stock_metrics_snapshot`

- snapshot_time
- symbol
- close
- volume
- rsi_14
- macd
- sma_20
- sma_50
- sma_200
- return_1m
- return_3m
- return_6m
- pe_ratio
- pb_ratio
- roe
- debt_to_equity
- revenue_growth
- earnings_growth
- score_total

---

## 4) Stock Scoring Logic (Simple & Effective)

Use weighted scoring so every stock gets a normalized score (0 to 100).

### Suggested Factors

- **Momentum (30%)**
  - 1M/3M returns, price above SMA50/SMA200
- **Quality (25%)**
  - ROE, low debt-to-equity, earnings growth
- **Valuation (20%)**
  - PE and PB relative to sector medians
- **Volatility/Risk (15%)**
  - Lower drawdown, stable ATR
- **Liquidity (10%)**
  - Average traded value

### Formula

```python
score_total = (
    0.30 * momentum_score +
    0.25 * quality_score +
    0.20 * valuation_score +
    0.15 * risk_score +
    0.10 * liquidity_score
)
```

Normalize each sub-score before weighting.

---

## 5) Example Pipeline (Python Pseudocode)

```python
from datetime import datetime
import pandas as pd

NIFTY50 = ["RELIANCE", "TCS", "HDFCBANK", "INFY", "ICICIBANK"]  # extend to full list


def fetch_prices(symbol):
    # Replace with production-grade source
    # Return dataframe with OHLCV
    pass


def compute_indicators(df):
    # RSI, MACD, SMA20/50/200, returns, volatility
    return df


def compute_fundamentals(symbol):
    # PE, PB, ROE, debt/equity, growth metrics
    return {
        "pe_ratio": None,
        "pb_ratio": None,
        "roe": None,
        "debt_to_equity": None,
        "revenue_growth": None,
        "earnings_growth": None,
    }


def score_stock(metrics):
    # Convert to normalized factor scores and combine
    return 0.0


rows = []
for symbol in NIFTY50:
    px = fetch_prices(symbol)
    ind = compute_indicators(px)
    fundamentals = compute_fundamentals(symbol)

    latest = {
        "snapshot_time": datetime.utcnow(),
        "symbol": symbol,
        "close": ind["close"].iloc[-1],
        "volume": ind["volume"].iloc[-1],
        **fundamentals,
    }
    latest["score_total"] = score_stock(latest)
    rows.append(latest)

snapshot_df = pd.DataFrame(rows).sort_values("score_total", ascending=False)
print(snapshot_df.head(10))
```

---

## 6) Streamlit UI Sections

### A. Universe & Filters

- Sector filter
- Market-cap/volatility filter
- Score threshold slider

### B. Top Ranked Stocks

- Rank, symbol, score, valuation, momentum signals
- Color-coded flags (green/yellow/red)

### C. Stock Detail View

- Candlestick chart with SMA overlays
- RSI + MACD panels
- Fundamental trend cards
- Recent news headlines (optional)

### D. Backtest Panel

- Rebalance frequency (weekly/monthly)
- Top N selection (5/10/15)
- Benchmark vs Nifty50

---

## 7) Minimal Streamlit App Skeleton

```python
import streamlit as st
import pandas as pd

st.set_page_config(page_title="Nifty50 Screener", layout="wide")
st.title("Nifty50 Smart Investment Dashboard")

# Load latest computed snapshot from DB/CSV
# df = load_snapshot()
df = pd.DataFrame([
    {"symbol": "RELIANCE", "score_total": 82.4, "pe_ratio": 24.1, "return_3m": 8.2},
    {"symbol": "TCS", "score_total": 79.8, "pe_ratio": 29.5, "return_3m": 6.9},
])

min_score = st.sidebar.slider("Minimum Score", 0, 100, 70)
filtered = df[df["score_total"] >= min_score].sort_values("score_total", ascending=False)

st.subheader("Top Picks")
st.dataframe(filtered, use_container_width=True)

selected = st.selectbox("Inspect Stock", filtered["symbol"].tolist() if not filtered.empty else [])
if selected:
    st.write(f"Detailed charts and analytics for {selected} go here.")
```

---

## 8) Practical Enhancements

1. Add regime filter (bull/bear/sideways) to avoid false positives.
2. Use sector-relative scoring (banks vs IT vs FMCG).
3. Add stop-loss and position-sizing suggestions.
4. Add explainability: why a stock got its score.
5. Add Telegram/email alert when score crosses threshold.

---

## 9) Validation Checklist

- Verify all Nifty50 symbols map correctly to your data provider.
- Detect stale/missing data before scoring.
- Cap influence of any single factor to avoid overfitting.
- Backtest across multiple market regimes.
- Compare turnover and transaction cost impact.

---

## 10) Suggested Build Sequence (Fastest Path)

1. Build data fetch + indicator computation for Nifty50 list.
2. Add scoring and produce daily ranked table.
3. Build Streamlit dashboard on top of ranked table.
4. Add backtest module.
5. Add alerts and deployment.

If you want, the next step can be a production-ready starter with:

- `data_pipeline.py`
- `scoring.py`
- `app.py`
- `requirements.txt`
- `.env.example`

