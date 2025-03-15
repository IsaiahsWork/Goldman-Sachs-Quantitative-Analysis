# Goldman-Sachs-Quantitative-Analysis

🔹Is Goldman Sachs (GS) a strong investment compared to the market? I conducted a quantitative SQL analysis of GS's Sharpe Ratio, RSI, 50-day & 200-day moving averages, and market correlation with S&P 500 (SPY) to assess its risk-adjusted performance. Here’s what I found:

📌 Key Findings from GS Stock Analysis
📅 Timeframe Analyzed: March 2020 – March 2025

🔹 Sharpe Ratio Analysis (Risk-Adjusted Return)
✔ What it is: Measures how much excess return an investment generates per unit of risk. 

Used the WITH function to Store results in a Common Table Expression (CTE) named Returned_Data, then used LAG() to fetch the previous day’s closing price and compute daily percentage returns. Finally, I calculated the Sharpe Ratio in the final query by using the AVG function in the numerator to calculates the average daily return and STDEV function in denominator to measure return volatility (risk)

Sharpe Ratio Query:
``
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
	FROM Returned_Data``

✔ Findings:

Lowest (-0.45) on March 18, 2020 → Extreme market uncertainty during COVID-19 crash.
Peak (~0.12) in April 2021 → Strong economic rebound.
Recent Stability (~0.06) in 2025 → Moderate risk-adjusted returns.

📌 Implication: GS has moved from extreme risk to a more stable investment option over time.

🔹 RSI (Relative Strength Index – Momentum Indicator)
✔ What it is: RSI measures the speed and magnitude of price movements to identify overbought or oversold conditions (scale from 0 to 100). 

The WITH function in SQL is used to create (CTEs), which helped organize complex queries like storing daily returns (Price_Change) for later calculations the using the previous CTE (Price_Changes) to calculate Average Gain & Average Loss.


RSI Query:

``WITH Price_Changes AS (
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
	ORDER BY Date DESC;``

✔ Findings:

Overbought (RSI > 70) on Feb 3, 2025: RSI 81.95, signaling high buying momentum.
Oversold (RSI < 30) on March 10, 2025: RSI 15.01, indicating potential buying opportunity.
Recent RSI (March 13, 2025): RSI 20.1, still in oversold territory.

📌 Implication: GS was overbought in early 2025 but is now oversold, suggesting a potential reversal.

🔹 GS vs. S&P 500: Market Performance Comparison
✔ What it is: Measures how GS performed relative to the overall market (S&P 500). 

Uses LAG() to fetch previous day’s closing price for both SPY and GS to computes their respective daily return as (Current Price - Previous Price) / Previous Price. Then I joined both tables by dates which ensures both GS and SPY returns are aligned on the same trading days.


Stock and Market Comparison Query:

``SELECT 
GS.Date,
GS.Close_Last AS GS_Close,
(GS.Close_Last - LAG(GS.Close_Last) OVER (ORDER BY GS.Date)) / LAG(GS.Close_Last) OVER (ORDER BY GS.Date) AS Stock_Return,
SPY.Close_Last AS SPY_Close,
(SPY.Close_Last - LAG(SPY.Close_Last) OVER (ORDER BY SPY.Date)) / LAG(SPY.Close_Last) OVER (ORDER BY SPY.Date) AS Market_Return
FROM SqlProjects.dbo.GS
JOIN SqlProjects.dbo.SPY ON GS.Date = SPY.Date
ORDER BY Stock_Return DESC;``

✔ Findings:

GS outperformed SPY in 2021 and 2023, indicating strong relative strength.
GS had a 13.09% return vs. SPY’s 2.48% on Nov 6, 2024, showing it delivered higher gains but with higher risk.
GS was more volatile than SPY, making it riskier in downturns.

📌 Implication: GS tends to amplify market movements—offering higher returns in bull markets but larger losses in bear markets.

🔹 50-Day vs. 200-Day Moving Average (MA) – Trend Strength
✔ What it is: Moving averages smooth price fluctuations and help identify trends.

50-Day MA: Short-term trend indicator.
200-Day MA: Long-term trend indicator. 

I used the AVG() window function to compute a rolling average of Close_Last (closing stock price). Used the functions ORDER BY Date ROWS BETWEEN to identify time frame for both the 50- and 200-day moving average.

Moving Average Query:

``SELECT 
Date,
Close_Last,
AVG(Close_Last) OVER (ORDER BY Date ROWS BETWEEN 49 PRECEDING AND CURRENT ROW) AS Moving_Avg_50,
AVG(Close_Last) OVER (ORDER BY Date ROWS BETWEEN 199 PRECEDING AND CURRENT ROW) AS Moving_Avg_200
FROM SqlProjects.dbo.GS
ORDER BY Moving_Avg_200 DESC;``


✔ Findings:

Golden Cross (June 2021): 50-day MA crossed above 200-day MA, signaling a strong bullish trend.
Death Cross (March 2022): 50-day MA fell below 200-day MA, signaling a bearish trend.
Recent Trend (2025): 50-day MA converging toward 200-day MA, suggesting potential consolidation or breakout.

📌 Implication:

Golden Cross → Buy signal (uptrend).
Death Cross → Sell signal (downtrend).
Current trend suggests a potential breakout.

🔍 Investment Insights from the Analysis
✔ GS was highly volatile during the 2020 market crash, with negative Sharpe Ratios. 

✔ GS outperformed the market in bullish periods but was more volatile in downturns. 

✔ RSI suggests GS is currently oversold, possibly a buy opportunity. 

✔ Golden Cross and Death Cross confirmed key trend shifts, with a new trend possibly forming.
