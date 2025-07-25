# 📊 Goldman Sachs Quantitative Analysis (2020–2025)

Can Goldman Sachs (GS) outperform the market? This SQL-driven project analyzes GS stock performance using key financial indicators like Sharpe Ratio, RSI, Moving Averages, and correlation with the S\&P 500 (SPY) to evaluate its risk-adjusted returns over a 5-year period.

---

## 📅 Timeframe

**March 2020 – March 2025**

## 📦 Data Overview

Daily closing price and volume data for:

* **GS (Goldman Sachs)**
* **SPY (S\&P 500 ETF)**

---

## 🔍 Analysis Objectives

✔️ **Sharpe Ratio** – Risk-adjusted return tracking
✔️ **RSI (Relative Strength Index)** – Overbought/oversold signal detection
✔️ **50/200-Day Moving Averages** – Trend momentum indicators
✔️ **Market Correlation** – GS vs. S\&P 500 behavior

---

## 📈 Sharpe Ratio Analysis

### ✅ Query Used:

```sql
WITH Returned_Data AS (
  SELECT
    Date,
    Volume,
    Close_Last,
    (Close_Last - LAG(Close_Last) OVER (ORDER BY DATE)) / LAG(Close_last) OVER (ORDER BY Date) AS Daily_Return
  FROM SqlProjects.dbo.GS
)
SELECT
  Date,
  Volume,
  Close_Last,
  Daily_Return,
  (AVG(Daily_Return) OVER (ORDER BY Date)) / (STDEV(Daily_Return) OVER (ORDER BY Date)) AS Sharpe_Ratio
FROM Returned_Data;
```

### 📊 Results:

* 📉 **-0.45** on March 18, 2020 → COVID-19 crash
* 📈 **\~0.12** in April 2021 → Rebound peak
* 📊 **0.06** in 2025 → Moderate stability

📌 **Implication:** GS became more stable over time as risk-adjusted returns improved.

---

## 📊 RSI (Relative Strength Index)

### ✅ Query Used:

```sql
WITH Price_Changes AS (
  SELECT
    Date,
    Volume,
    Close_Last,
    (Close_Last - LAG(Close_Last) OVER (ORDER BY DATE)) / LAG(Close_last) OVER (ORDER BY Date) AS Price_Change
  FROM SqlProjects.dbo.GS
),
Avg_Gains_Losses AS (
  SELECT
    Date,
    Volume,
    Close_Last,
    AVG(CASE WHEN Price_Change > 0 THEN Price_Change ELSE 0 END) OVER (ORDER BY Date ROWS BETWEEN 13 PRECEDING AND CURRENT ROW) AS Avg_Gain,
    AVG(CASE WHEN Price_Change < 0 THEN ABS(Price_Change) ELSE 0 END) OVER (ORDER BY Date ROWS BETWEEN 13 PRECEDING AND CURRENT ROW) AS Avg_Loss
  FROM Price_Changes
)
SELECT
  Date,
  Volume,
  Close_Last,
  100 - (100 / (1 + (Avg_Gain / NULLIF(Avg_Loss, 0)))) AS RSI
FROM Avg_Gains_Losses
ORDER BY Date DESC;
```

### 📊 Results:

* **RSI 81.95** → Feb 3, 2025 → Overbought
* **RSI 15.01** → Mar 10, 2025 → Oversold
* **RSI 20.1** → Mar 13, 2025 → Still oversold

📌 **Implication:** Signals potential reversal; GS moved from overbought to oversold.

---

## 🔄 Market Performance vs. S\&P 500

### ✅ Query Used:

```sql
SELECT
  GS.Date,
  GS.Close_Last AS GS_Close,
  (GS.Close_Last - LAG(GS.Close_Last) OVER (ORDER BY GS.Date)) / LAG(GS.Close_Last) OVER (ORDER BY GS.Date) AS Stock_Return,
  SPY.Close_Last AS SPY_Close,
  (SPY.Close_Last - LAG(SPY.Close_Last) OVER (ORDER BY SPY.Date)) / LAG(SPY.Close_Last) OVER (ORDER BY SPY.Date) AS Market_Return
FROM SqlProjects.dbo.GS
JOIN SqlProjects.dbo.SPY ON GS.Date = SPY.Date
ORDER BY Stock_Return DESC;
```

### 📊 Results:

* **Outperformed** in 2021 & 2023
* **Nov 6, 2024:** GS = **13.09%**, SPY = **2.48%**
* More **volatile** than market overall

📌 **Implication:** GS amplifies market direction—greater returns in bull markets, steeper losses in bear markets.

---

## 📉 50-Day vs. 200-Day Moving Average

### ✅ Query Used:

```sql
SELECT
  Date,
  Close_Last,
  AVG(Close_Last) OVER (ORDER BY Date ROWS BETWEEN 49 PRECEDING AND CURRENT ROW) AS Moving_Avg_50,
  AVG(Close_Last) OVER (ORDER BY Date ROWS BETWEEN 199 PRECEDING AND CURRENT ROW) AS Moving_Avg_200
FROM SqlProjects.dbo.GS
ORDER BY Moving_Avg_200 DESC;
```

### 📊 Results:

* **Golden Cross** → June 2021 → Bullish trend
* **Death Cross** → March 2022 → Bearish signal
* **2025** → MAs converging → Consolidation or breakout expected

📌 **Implication:**

* Golden Cross → Buy signal
* Death Cross → Sell signal
* Now: market indecision, but preparing for next move

---

## 🧠 Investment Insights Summary

✔ Volatile in early 2020; stabilized post-COVID crash
✔ Outperforms market during uptrends; underperforms in downturns
✔ RSI suggests possible buy opportunity
✔ Trend signals (Golden/Death Cross) confirmed momentum changes

---

## 📢 Final Thoughts

This SQL-based equity performance study shows how data analysis can inform investment decisions. By comparing GS to macro trends and using technical indicators, investors can better understand risk-reward dynamics over time.

**Let’s connect** – [LinkedIn](https://www.linkedin.com/in/isaiah-l-wright/) | [Portfolio](https://isaiahlaruewright.wixsite.com/isaiahswork)
