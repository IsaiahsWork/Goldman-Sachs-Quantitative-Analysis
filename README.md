# 📈 Goldman Sachs Quantitative Analysis (2020–2025)

🔹 Is Goldman Sachs (GS) a strong investment compared to the market?

I conducted a quantitative SQL analysis of GS's Sharpe Ratio, RSI, 50-day & 200-day moving averages, and market correlation with S\&P 500 (SPY) from **March 2020 – March 2025** to assess its risk-adjusted performance. I collected the data from Yahoo Finance.

---

## 📦 Data Structure Overview

| Column Name   | Description                                       |
| ------------- | ------------------------------------------------- |
| Date          | Trading day (business calendar)                   |
| Volume        | Number of shares traded on the day                |
| Close\_Last   | Final traded price of the day                     |
| Daily\_Return | Percentage change in closing price from prior day |

SPY and GS datasets followed the same schema, joined via `Date`.

---

## 🎯 Analysis Objectives

✔️ Evaluate GS's **risk-adjusted returns** using the Sharpe Ratio
✔️ Detect **momentum trends** with RSI
✔️ Compare GS's **performance relative to S\&P 500 (SPY)**
✔️ Measure **trend strength** using 50-day and 200-day moving averages
✔️ Translate SQL analytics into investment insights

---

## 🔧 Methods Used

* Common Table Expressions (CTEs)
* Window Functions: `LAG()`, `AVG()`, `STDEV()`, `ROWS BETWEEN`
* Time-Series SQL Analysis
* Join operations for cross-symbol comparisons

---

## 📊 Sharpe Ratio Analysis (Risk-Adjusted Return)

✔ **What it is**: Measures how much excess return an investment generates per unit of risk.
Used a CTE to calculate daily return using `LAG()` and then computed the Sharpe Ratio as average return divided by return volatility.

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

### 📊 Findings:

* 📉 Lowest (-0.45) on **March 18, 2020** → Extreme market uncertainty during COVID-19 crash.
* 📈 Peak (\~0.12) in **April 2021** → Strong economic rebound.
* 🟰 Recent Stability (\~0.06) in **2025** → Moderate risk-adjusted returns.

📌 **Implication**: GS has moved from extreme risk to a more stable investment option over time.

---

## 🔄 RSI (Relative Strength Index – Momentum Indicator)

✔ **What it is**: RSI measures the speed and magnitude of price movements to identify overbought or oversold conditions (scale from 0 to 100).

Two CTEs were used to calculate Price\_Change and then rolling averages of gains and losses.

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

### 📊 Findings:

* 📈 Overbought (RSI > 70) on **Feb 3, 2025**: RSI **81.95**
* 📉 Oversold (RSI < 30) on **March 10, 2025**: RSI **15.01**
* 📉 Recent (March 13, 2025): RSI **20.1**, still oversold

📌 **Implication**: GS was overbought in early 2025 but is now oversold, suggesting a potential reversal.

---

## 📈 GS vs. S\&P 500: Market Performance Comparison

✔ **What it is**: Measures how GS performed relative to the overall market (S\&P 500).

Daily returns were calculated using `LAG()` for both GS and SPY and then joined by date.

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

### 📊 Findings:

* ✅ GS outperformed SPY in **2021** and **2023**
* 📅 On **Nov 6, 2024**: GS +13.09% vs. SPY +2.48%
* ⚠️ GS was more volatile than SPY

📌 **Implication**: GS amplifies market trends—higher gains in bull markets, steeper losses in bear markets.

---

## 📊 50-Day vs. 200-Day Moving Averages – Trend Strength

✔ **What it is**: Moving averages smooth price action and identify long/short-term trends.

Used `AVG()` window functions with `ROWS BETWEEN` for moving average calculations.

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

### 📊 Findings:

* 📈 **Golden Cross** (June 2021): Bullish signal
* 📉 **Death Cross** (March 2022): Bearish signal
* 🟰 **2025**: MAs converging—possible breakout ahead

📌 **Implication**: Trend structure suggests upcoming breakout; keep an eye on moving averages.

---

## 🔍 Investment Insights from the Analysis

✔ GS was highly volatile during the 2020 market crash, with negative Sharpe Ratios.
✔ GS outperformed the market in bullish periods but was more volatile in downturns.
✔ RSI suggests GS is currently oversold, possibly a buy opportunity.
✔ Moving Averages confirmed key trend shifts, with a new trend possibly forming.

---

📁 Files and further work can be found on my [Portfolio]([https://github.com](https://isaiahlaruewright.wixsite.com/isaiahswork)) or [LinkedIn]([https://linkedin.com](https://www.linkedin.com/in/isaiah-l-wright/)). Thank you for reading!
