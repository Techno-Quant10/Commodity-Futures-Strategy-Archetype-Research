# Commodity Futures - Strategy Archetype Research

### How trend following, mean reversion and volatility breakout behave across five commodity futures markets, 2000-2026

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange)
![Type](https://img.shields.io/badge/Type-Quantitative%20Research-lightgrey)
![License](https://img.shields.io/badge/License-MIT-green)

![Final research dashboard](figures/09_final_research_dashboard.png)

*Final research dashboard: expectancy, Profit Factor, win rate and average holding period for each commodity x strategy combination (2,499 frozen trades).*

> **Nature of this project.** This is a research study of *strategy archetype behaviour*. It is not an optimised trading system, not a live strategy, and not a statement about future profitability. All results are historical, trade-level, and gross of implementation costs. Observed historical results are kept strictly separate from statistically validated conclusions throughout.

---

## Project at a Glance

| Item | Value |
| --- | --- |
| Commodity futures | 5 (Gold, Silver, Copper, WTI Crude Oil, Henry Hub Natural Gas) |
| Strategy archetypes | 3 (MACD Trend Following, Hurst + Z-Score Mean Reversion, Bollinger Breakout) |
| Commodity x strategy combinations | 15 |
| Frozen trade archive | 2,499 trades |
| Raw daily market observations | 32,662 |
| Common analytical sample | 2000-08-30 to 2026-09-04 (about 26.01 years) |
| Historical sub-periods | 5 |
| Combined market states | 7 (regime-state coverage about 97.68%) |
| Bootstrap | 10,000 iterations, seed 42 |
| Conditional tests / directional comparisons | 240 / 82 |
| Duplicate market observations | 0 |
| Missing OHLCV observations | 0 |
| Extreme-trade validation flags | 0 |
| Data source | Yahoo Finance continuous futures series |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Research Question and Unit of Analysis](#2-research-question-and-unit-of-analysis)
3. [How to Read the Results: Observed vs Statistically Validated](#3-how-to-read-the-results-observed-vs-statistically-validated)
4. [Research Pipeline](#4-research-pipeline)
5. [Data](#5-data)
6. [Common Trade Engine](#6-common-trade-engine)
7. [Strategy Specifications](#7-strategy-specifications)
8. [Metrics and Definitions](#8-metrics-and-definitions)
9. [Results: Commodity x Strategy Performance](#9-results-commodity-x-strategy-performance)
10. [Results: Strategy Archetype Comparison](#10-results-strategy-archetype-comparison)
11. [Results: Distributions and Mean vs Median](#11-results-distributions-and-mean-vs-median)
12. [Results: Long vs Short Asymmetry](#12-results-long-vs-short-asymmetry)
13. [Results: Tail Concentration, Trimming and Extreme Trades](#13-results-tail-concentration-trimming-and-extreme-trades)
14. [Results: Temporal Stability](#14-results-temporal-stability)
15. [Results: Market-State (Regime) Analysis](#15-results-market-state-regime-analysis)
16. [Statistical Validation](#16-statistical-validation)
17. [Primary Results vs Robustness Diagnostics](#17-primary-results-vs-robustness-diagnostics)
18. [Descriptive Price Behaviour](#18-descriptive-price-behaviour)
19. [Final Research Dashboard](#19-final-research-dashboard)
20. [Key Findings](#20-key-findings)
21. [Interpretation: Why the Results May Look the Way They Do](#21-interpretation-why-the-results-may-look-the-way-they-do)
22. [What This Project Does Not Claim](#22-what-this-project-does-not-claim)
23. [Limitations](#23-limitations)
24. [Reproducibility and Archive](#24-reproducibility-and-archive)
25. [Repository Structure](#25-repository-structure)
26. [Future Research](#26-future-research)
27. [Data Licensing, Disclaimer and License](#27-data-licensing-disclaimer-and-license)

---

## 1. Executive Summary

This repository documents a quantitative study of how three systematic strategy archetypes behave across five major commodity futures markets over roughly 26 years of daily data. The study produced a frozen archive of **2,499 trades** across **15 commodity x strategy combinations** and analysed it through trade-level statistics, distributional and tail diagnostics, long/short decomposition, temporal stability, causal market-state analysis, and formal statistical testing (bootstrap confidence intervals and Benjamini-Hochberg false discovery rate control).

The objective is **not** to identify the single highest-return strategy. The objective is to understand how strategy mechanics interact with each commodity's return distribution, volatility, directional behaviour, tail characteristics and stability through time.

### Headline observations

| # | Observation | Observed value | Evidence status |
| --- | --- | --- | --- |
| 1 | Strategy performance depends on the commodity | MACD expectancy: Silver +1.83%, Crude Oil +1.01%, Copper +0.85%, Gold +0.73%, Natural Gas -2.27% | Observed. No positive combination survived FDR correction |
| 2 | Win rate does not equal profitability | Mean Reversion: 63.98% win rate, mean -0.0558%, Profit Factor 0.9788 | Observed (pooled statistic) |
| 3 | Archetypes have opposite distribution shapes | Skew: MACD +3.16, Mean Reversion -2.79, Bollinger +2.05 | Observed |
| 4 | Direction matters descriptively | Silver MACD long +3.18% vs short -0.42%; Gold MACD long +1.97% vs short -0.89% | Observed. 0 of 82 directional comparisons significant after FDR |
| 5 | Silver MACD is the strongest observed cell | +1.83% expectancy, PF 1.62, win rate 37.42%, median -2.09% | Observed. Not statistically established |
| 6 | Only negative bootstrap evidence: two Natural Gas cells | NG MACD 95% CI about [-4.30%, -0.13%]; NG Bollinger about [-3.63%, -0.07%] | Tested (bootstrap) |
| 7 | Regime patterns are interesting but unproven | e.g. Copper MACD in uptrend + high volatility about +5.83% | 0 best-vs-worst state comparisons significant at 5% FDR |
| 8 | Conditional results are partly tail-sensitive | Direction preserved in 80.70% (5% trim) and 70.18% (10% trim) of comparisons | Diagnostic |

The central methodological conclusion is that **no positive commodity x strategy combination survived multiple-testing correction at the 5% level**. Positive observed expectancies are therefore reported as historical observations, not as evidence of a statistically proven edge.

---

## 2. Research Question and Unit of Analysis

**Central question:** *How does the interaction between strategy mechanics and the statistical characteristics of different commodity futures affect trading outcomes?*

The project deliberately avoids treating "strategy" as the unit of analysis. The analytical unit is:

```
Strategy x Commodity x Direction x Distribution x Time x Market State
```

Each dimension is examined explicitly:

| Dimension | What is examined |
| --- | --- |
| Strategy | MACD Trend Following, Mean Reversion, Bollinger Breakout |
| Commodity | Gold, Silver, Copper, Crude Oil, Natural Gas |
| Direction | Long vs short trades, separately |
| Distribution | Mean, median, standard deviation, skewness, excess kurtosis, payoff, Profit Factor |
| Tail behaviour | Profit and loss concentration, trimming diagnostics, extreme trades |
| Time | Five fixed historical sub-periods |
| Market state | Causal trend, momentum and volatility classification |
| Statistical evidence | t-statistics, bootstrap confidence intervals, FDR-adjusted p-values |

---

## 3. How to Read the Results: Observed vs Statistically Validated

Every claim in this README belongs to one of four evidence classes:

| Class | Meaning |
| --- | --- |
| **Observed** | A descriptive statistic computed from the frozen trade archive. It describes what happened in this sample. |
| **Tested** | A result from a formal statistical procedure (t-test, bootstrap, FDR correction). |
| **Diagnostic** | A robustness check (for example trimming) used to understand tail dependence. It never replaces primary results. |
| **Interpretation** | An economic or statistical explanation offered to aid understanding. It is not tested by this project. |

Wording convention used throughout:

| Acceptable | Not acceptable |
| --- | --- |
| "Silver MACD produced the highest observed average trade expectancy of +1.83%." | "Silver MACD is statistically proven to have a +1.83% edge." |
| "Regime analysis identified economically interesting conditional patterns." | "The regime filter proves when the strategy works." |
| "Natural Gas Mean Reversion produced +0.63% historical expectancy." | "Natural Gas is a mean-reverting market." |

---

## 4. Research Pipeline

The repository is organised as one coherent pipeline rather than a set of independent notebooks:

```
Data Acquisition (Yahoo Finance continuous futures)
        |
        v
Data Validation (duplicates, missing OHLCV, gaps, non-positive prices)
        |
        v
Research Universe (5 commodities, common sample 2000-08-30 to 2026-09-04)
        |
        v
Strategy Implementation
        |-- MACD Trend Following
        |-- Hurst Exponent + Z-Score Mean Reversion
        +-- Bollinger Band Breakout
        |
        v
Trade Generation (common execution engine, 2,499 trades)
        |
        v
Trade-Level Statistics --> Distribution Analysis --> Tail Analysis
        |
        v
Temporal Stability --> Causal Regime Analysis
        |
        v
Statistical Validation (t-test, bootstrap, BH-FDR)
        |
        v
Extreme Trade Validation (0 flags)
        |
        v
Frozen Research Archive
        |
        v
Visualization --> Final Dashboard --> Research Conclusions
```

The visualization stage reads from the frozen archive. It does not rerun strategies, download new data, optimise parameters, alter trades, winsorise primary returns, or cap returns.

---

## 5. Data

### 5.1 Research universe

| Commodity | Exchange | Yahoo Finance ticker | Raw rows | First observation |
| --- | --- | --- | ---: | --- |
| Gold | COMEX | GC=F | 6,528 | 2000-08-30 |
| Silver | COMEX | SI=F | 6,530 | 2000-08-30 |
| Copper | COMEX | HG=F | 6,533 | 2000-08-30 |
| WTI Crude Oil | NYMEX | CL=F | 6,537 | 2000-08-23 |
| Henry Hub Natural Gas | NYMEX | NG=F | 6,534 | 2000-08-30 |
| **Total** | | | **32,662** | |

The project originally considered MCX-listed commodities. The final study uses international futures because the historical Yahoo Finance futures data were more suitable for the intended research. Cotton was explicitly excluded from the final universe.

### 5.2 Common analytical sample

**Common sample: 2000-08-30 to 2026-09-04 (about 26.01 years).** The common sample starts at the latest first observation among the five series (Crude Oil begins seven days earlier, on 2000-08-23).

A common sample matters for cross-commodity comparison because all five markets are then evaluated across the same calendar window and therefore the same macroeconomic episodes. Differences in outcomes across commodities cannot be attributed to markets being observed over different eras, and the five historical sub-periods used in the temporal analysis line up across all 15 cells.

### 5.3 Data quality audit

| Check | Result |
| --- | --- |
| Duplicate observations | 0 |
| Missing OHLCV observations | 0 |
| Median observation gap | 1 day |
| Maximum observation gap | 5 days |
| Gaps greater than 7 days | 0 |
| Common-sample coverage | Common sample defined: 2000-08-30 to 2026-09-04 |
| Non-positive prices | Present in one event (WTI, April 2020); handled explicitly (see 5.6) |

Why these checks matter for systematic backtesting:

- **Duplicate dates** can double-count bars, distort rolling windows and produce artificial repeated signals.
- **Missing OHLCV values** break rolling-window indicators (EMA, SMA, rolling Hurst, channel highs and lows) and can silently shift lookback windows.
- **Calendar continuity and gap size** reveal stale or missing data. A maximum gap of 5 days is the size expected around weekends and exchange holidays; there are no multi-week gaps.
- **Non-positive prices** break percentage-return arithmetic and must be treated deliberately rather than silently.

### 5.4 Continuous futures caveat

The Yahoo Finance symbols represent **continuous futures series**. This project does **not** reconstruct individual futures contracts and does **not** implement a custom contract-roll methodology. The strategies are tested on the supplied continuous price series.

| Concept | Meaning | Status in this project |
| --- | --- | --- |
| Continuous futures series | A single price series spliced from successive contracts | Used as supplied |
| Individual contracts | A specific expiry with its own price history | Not reconstructed |
| Roll methodology | The rule and price adjustment used when moving from one contract to the next | Not implemented or controlled here |
| Actual tradable futures P&L | Profit and loss of a position held in specific contracts, including roll yield, costs and margin | Not modelled |

Consequence: a continuous series can contain discontinuities at roll dates that reflect the price spread between contracts rather than a market return. The exact splicing and adjustment convention of the Yahoo series is outside this project's control. Results describe strategy behaviour on that series and **must not be described as exchange-level historical futures P&L**.

### 5.5 Daily return characteristics

Before any strategy testing, daily return distributions were examined.

| Commodity | Mean | Median | Std. Dev. | Minimum | Maximum |
| --- | ---: | ---: | ---: | ---: | ---: |
| Gold | +0.0488% | +0.0501% | 1.1267% | -11.37% | +9.03% |
| Silver | +0.0616% | +0.1038% | 2.0859% | -31.35% | +14.03% |
| Copper | +0.0455% | +0.0276% | 1.7139% | -22.25% | +13.25% |
| Crude Oil | +0.0505% | +0.1089% | 2.6121% | -24.59% | +25.10% |
| Natural Gas | +0.0666% | -0.0312% | 3.8556% | -47.48% | +46.48% |

**Volatility ordering:** Natural Gas > Crude Oil > Silver > Copper > Gold. Natural Gas daily standard deviation (about 3.86%) is roughly 3.4 times that of Gold (about 1.13%).

Approximate scale (daily standard deviation multiplied by sqrt(252), a standard annualisation convention applied here for orientation only):

| Commodity | Daily SD | Approx. annualised | Multiple of Gold |
| --- | ---: | ---: | ---: |
| Gold | 1.13% | about 18% | 1.0x |
| Copper | 1.71% | about 27% | 1.5x |
| Silver | 2.09% | about 33% | 1.9x |
| Crude Oil | 2.61% | about 41% | 2.3x |
| Natural Gas | 3.86% | about 61% | 3.4x |

Why this matters for strategy design and interpretation:

1. **Fixed thresholds mean different things in different markets.** A regime threshold of +/-5% from the 200-day average is about 4.4 daily standard deviations for Gold but only about 1.3 for Natural Gas. The same rule set is effectively "wider" in calm markets and "narrower" in volatile ones.
2. **Identical indicator settings are not volatility-normalised.** MACD 12/26/9, 2-standard-deviation bands and a 20-day channel exit are applied unchanged to every market.
3. **Trade-return dispersion scales with, but does not map one-for-one onto, daily volatility.** For example, Silver MACD has the highest trade standard deviation (15.23%) even though Natural Gas has higher daily volatility.
4. **Daily returns are heavy-tailed.** Natural Gas ranges from -47.48% to +46.48% in a single day, while the daily mean is under 0.07%. Drift is tiny relative to dispersion.

### 5.6 Crude Oil negative-price event (April 2020)

The historical data contain the April 2020 WTI negative-price event:

| Date | Field | Value |
| --- | --- | ---: |
| 2020-04-20 | Close | -37.63 |
| 2020-04-20 | Low | about -40.32 |
| 2020-04-21 | Open | about -14.00 |
| 2020-04-21 | Close | +10.01 |

**Why negative prices occurred (market background, not derived from this dataset).** The front-month contract was days from expiry, storage capacity for physical delivery was constrained, and holders of long positions who could not take delivery were forced sellers. The contract settled below zero.

**Why percentage-return formulas fail around zero.** The simple return is (P_t - P_{t-1}) / P_{t-1}. When the denominator is negative the sign of the return inverts, and when prices cross zero the result is not economically meaningful. Using the dataset's own values, the naive close-to-close return on 2020-04-21 would be (10.01 - (-37.63)) / (-37.63) = -126.6%, reporting a price *rise* as a large *loss*.

**Why the observation is not deleted.** It is a genuine historical market event. Removing it would silently edit history, hide an important tail regime, and make the dataset look cleaner than the market was.

**How the engine handles it.** Non-positive prices are treated as **invalid execution prices** by the strategy engine, and invalid execution prices are skipped. No trade is executed at a non-positive price.

**Related observations.** The reported Crude Oil daily-return extremes (-24.59% and +25.10%) are of ordinary magnitude, which implies that returns spanning the non-positive observations are not part of those descriptive statistics. In the descriptive cumulative price chart (Figure 1), the Crude Oil line falls below -100% around April 2020, which is only possible because the price itself is negative.

**Caveat.** This treatment prevents invalid arithmetic and invalid executions. It is **not** a complete economic treatment of trading negative-price futures, and it remains an important data caveat for the Crude Oil results.

---

## 6. Common Trade Engine

All three strategies share one research execution framework.

| Rule | Specification |
| --- | --- |
| Signal timing | Signal computed from information available at the close of day t |
| Entry | Next trading day's **Open** (day t+1) |
| Directions | Long and short both supported |
| Opposite signals | Do **not** automatically reverse an open position |
| Position lifecycle | A position stays open until its own exit condition occurs |
| New trades | A new trade can start only after the previous position exits |
| Invalid prices | Non-positive execution prices are skipped |

Trade return formulas (per trade, in percent of entry price, no leverage, no compounding):

```math
R_{\text{long}} = \frac{P_{\text{exit}} - P_{\text{entry}}}{P_{\text{entry}}}
\qquad
R_{\text{short}} = \frac{P_{\text{entry}} - P_{\text{exit}}}{P_{\text{entry}}}
```

Each trade record contains:

| Field | Description |
| --- | --- |
| Ticker | Yahoo Finance symbol |
| Strategy | MACD / Mean Reversion / Bollinger |
| Entry Date | Date of entry execution |
| Exit Date | Date of exit |
| Position | Long or Short |
| Entry Price | Execution price at entry |
| Exit Price | Execution price at exit |
| Return % | Trade return per the formulas above |
| Holding Days | Duration of the trade |

**Why next-day execution.** The signal is computed from the day-t bar, whose close is only known once the session ends. Assuming execution at that same close would assume the strategy could act on a price that was itself used to generate the signal. Executing at the next open is the first price realistically available after the information is known. It also lets the close-to-open gap, which can be adverse, flow into trade returns rather than being assumed away.

**Why opposite signals do not reverse positions.** The strategies are not stop-and-reverse systems. An opposite entry signal that occurs while a position is open is ignored until the position's own exit rule fires. This keeps each trade's lifecycle well defined and makes holding periods and exit behaviour a property of the exit rule alone.

---

## 7. Strategy Specifications

### 7.1 Parameter summary

| Strategy | Parameters |
| --- | --- |
| MACD Trend Following | EMA 12 / 26 / 9; exit on 20-day channel break |
| Hurst + Z-Score Mean Reversion | Rolling Hurst window 252, maximum lag 100; Z-score lookback 20; entry when abs(Z) > 2 with H < 0.5; exit at Z crossing 0 |
| Bollinger Breakout | 20-day SMA and standard deviation, 2-standard-deviation bands, 200-day SMA trend filter, 10-day channel exit |

These are fixed specification values. They are not presented as optimised or tuned parameters.

### 7.2 Strategy 1: MACD Trend Following

```math
\text{MACD}_t = \text{EMA}_{12}(P)_t - \text{EMA}_{26}(P)_t
\qquad
\text{Signal}_t = \text{EMA}_9(\text{MACD})_t
\qquad
\text{EMA}_n(P)_t = \alpha P_t + (1-\alpha)\,\text{EMA}_n(P)_{t-1},\ \ \alpha = \frac{2}{n+1}
```

| Element | Rule |
| --- | --- |
| Long entry | MACD crosses above the signal line; execute at next-day Open |
| Short entry | MACD crosses below the signal line; execute at next-day Open |
| Long exit | Low_t < lowest Low of the prior 20 days |
| Short exit | High_t > highest High of the prior 20 days |

The 20-day channel is **shifted by one bar** so that it is built from days t-20 through t-1. The current day's low or high therefore never contributes to the threshold that it is being compared against.

Economic logic of trend following, and where it shows up in the data:

| Feature | Explanation | Evidence in this project |
| --- | --- | --- |
| Trend persistence | The strategy profits only if moves continue after the crossover | Large winners in Silver, Natural Gas, Copper and Crude MACD (Section 13.3) |
| Asymmetric payoff | Losses are cut at the channel break; winners can run | MACD payoff ratio 1.839 |
| Low hit rate | Many crossovers occur in ranges and fail | MACD win rate 37.33%; Natural Gas 29.44% |
| Dependence on large winners | The mean is driven by the right tail | Silver MACD: top-tail winners produce 50.6% of gross profit |
| Delayed exits | A channel break exits after part of the trend has been given back | Average holding about 42 days |
| Directional exposure | Profits depend on sustained directional moves | Long-side expectancy exceeds short-side in all five MACD cells |

### 7.3 Strategy 2: Hurst Exponent + Z-Score Mean Reversion

**Hurst exponent intuition.** The Hurst exponent H describes how the dispersion of price changes scales with the time lag tau:

```math
\sigma(\tau) = \operatorname{std}\big(P_{t+\tau} - P_t\big) \;\propto\; \tau^{H}
```

| H | Interpretation |
| --- | --- |
| H = 0.5 | Random-walk-like scaling |
| H < 0.5 | Anti-persistent (mean-reverting) behaviour |
| H > 0.5 | Persistent (trending) behaviour |

The above is the conceptual definition; the notebook contains the exact estimator.

**Parameters:** rolling window = 252 observations, maximum lag = 100, Z-score lookback = 20 observations.

```math
Z_t = \frac{P_t - \mu_{20,t}}{\sigma_{20,t}}
```

| Element | Rule |
| --- | --- |
| Mean-reversion condition | H < 0.5 |
| Long entry | H < 0.5 AND Z < -2 |
| Short entry | H < 0.5 AND Z > +2 |
| Long exit | Z > 0 |
| Short exit | Z < 0 |
| Execution | Next trading day's Open |

Design notes:

- **Why H < 0.5 is the condition.** It is the theoretical boundary between persistent and anti-persistent scaling; the filter restricts entries to periods when recent price behaviour looks anti-persistent.
- **Why the implementation is causal.** The rolling Hurst value at day t is estimated only from the 252 observations ending at t. Nothing after t enters it.
- **Why full-sample Hurst estimation would create look-ahead bias.** A Hurst value estimated over the entire 26-year history would embed knowledge of future prices when deciding, say, that a market "was mean-reverting" in 2003. The strategy would be conditioned on information that was not available at the time.
- **Why Z-score extremes represent deviation from a recent reference level.** With a 20-observation lookback the Z-score measures how many recent standard deviations the price sits away from its recent average. |Z| > 2 flags an unusually large displacement relative to the last month or so of trading.
- **Estimation noise.** With a 252-observation window and lags up to 100, only about 2 to 3 non-overlapping observations exist at the longest lag, so H is a noisy estimate. A rolling H below 0.5 is not by itself statistical evidence of mean reversion, since a random walk's estimates also scatter around 0.5 in finite samples.
- **Structure of the exit.** The specified exit is Z-score based (return of Z through zero). No separate price stop is part of the specification. Gains are therefore structurally bounded by the distance back to the mean, while a displaced market can keep moving against the position (see Section 21.2).

### 7.4 Strategy 3: Bollinger Band Breakout

```math
\text{Upper}_t = \text{SMA}_{20,t} + 2\,\sigma_{20,t}
\qquad
\text{Lower}_t = \text{SMA}_{20,t} - 2\,\sigma_{20,t}
```

| Element | Rule |
| --- | --- |
| Long entry | (1) Price > Upper band; (2) previous price <= previous Upper band; (3) Price > SMA200 |
| Short entry | (1) Price < Lower band; (2) previous price >= previous Lower band; (3) Price < SMA200 |
| Long exit | Low_t < lowest Low of the prior 10 days |
| Short exit | High_t > highest High of the prior 10 days |
| Execution | Next trading day's Open |

Design notes:

- **Why the previous-band condition matters.** It converts "price is outside the band" (a state) into "price has just crossed the band" (an event). Without it, every day spent outside the band would qualify as a signal. Because a new trade can open right after an exit, a persistent outside-band state would trigger immediate re-entry and stale entries deep into extended moves. The condition isolates a fresh breakout.
- **Why the 200-day filter.** It restricts longs to prices above, and shorts to prices below, the long-term average, aligning breakouts with the broader trend.
- **Why a shorter exit than MACD.** The 10-day channel is tighter than the 20-day channel, so trades are held for a shorter time (about 25 days on average versus about 42 for MACD).

---

## 8. Metrics and Definitions

All figures are trade-level, in percent of entry price, gross of costs.

| Metric | Definition |
| --- | --- |
| Expectancy (mean trade return) | Arithmetic mean of trade returns |
| Median | Median trade return |
| Win rate (p) | Share of trades with return > 0 |
| Average win (W), average loss (L) | Mean return of winners; mean absolute return of losers |
| Payoff ratio (b) | W / L |
| Profit Factor (PF) | Gross profit / absolute gross loss |
| Standard deviation | Dispersion of trade returns |
| Skewness | Third standardised moment; sign shows tail asymmetry |
| Excess kurtosis | Fourth standardised moment minus 3; tail thickness relative to normal |
| Trimmed mean | Mean after removing the most extreme observations at a stated proportion (diagnostic only) |
| Profit concentration | Share of gross profits produced by the top tail of winning trades |
| Loss concentration | Share of gross losses produced by the bottom tail of losing trades |

Core identities used in the interpretation sections:

```math
E = pW - (1-p)L
\qquad
\text{PF} = \frac{p}{1-p}\,b
\qquad
p^{*} = \frac{1}{1+b}
```

Here p* is the **break-even win rate**: the win rate at which expectancy is exactly zero for a given payoff ratio. Expectancy is positive when the observed win rate exceeds p*.

```math
\text{Skewness} = \frac{E[(R-\mu)^3]}{\sigma^3}
\qquad
\text{Excess kurtosis} = \frac{E[(R-\mu)^4]}{\sigma^4} - 3
```

**Trade-level is not portfolio-level.** A +1.83% trade expectancy is not a +1.83% monthly or annual portfolio return. Portfolio outcomes require capital allocation, position sizing, leverage, margin, overlapping trades, contract multipliers, cash treatment, reinvestment and roll methodology. This project deliberately keeps trade-level statistical analysis separate from portfolio-level performance analysis.

---

## 9. Results: Commodity x Strategy Performance

![Commodity x strategy performance](figures/02_commodity_strategy_performance.png)

*Figure 2. Expectancy, Profit Factor, win rate and average holding period for the 5 x 3 commodity x strategy matrix.*

### 9.1 All 15 cells

| Commodity | Strategy | Trades | Win rate | Mean | Median | Std. Dev. | Avg hold (days) | PF |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Gold | MACD | 154 | 40.91% | +0.73% | -1.10% | 6.75% | 44.21 | 1.38 |
| Gold | Mean Reversion | 203 | 65.52% | +0.13% | +1.04% | 4.08% | 22.53 | 1.10 |
| Gold | Bollinger | 153 | 38.56% | -0.03% | -0.74% | 4.44% | 24.94 | 0.98 |
| Silver | MACD | 155 | 37.42% | +1.83% | -2.09% | 15.23% | 43.53 | 1.62 |
| Silver | Mean Reversion | 205 | 61.95% | -0.17% | +1.83% | 8.85% | 22.63 | 0.94 |
| Silver | Bollinger | 153 | 34.64% | -0.05% | -2.30% | 8.90% | 25.25 | 0.98 |
| Copper | MACD | 156 | 41.03% | +0.85% | -1.65% | 9.65% | 42.85 | 1.34 |
| Copper | Mean Reversion | 197 | 67.01% | -0.33% | +1.33% | 7.67% | 21.36 | 0.86 |
| Copper | Bollinger | 134 | 39.55% | +1.15% | -1.01% | 8.41% | 26.77 | 1.58 |
| Crude Oil | MACD | 148 | 39.19% | +1.01% | -2.97% | 12.59% | 45.49 | 1.27 |
| Crude Oil | Mean Reversion | 181 | 60.77% | -0.57% | +1.47% | 8.71% | 23.04 | 0.81 |
| Crude Oil | Bollinger | 141 | 41.13% | +0.12% | -2.00% | 9.60% | 25.26 | 1.04 |
| Natural Gas | MACD | 180 | 29.44% | -2.27% | -5.14% | 14.38% | 34.83 | 0.64 |
| Natural Gas | Mean Reversion | 194 | 64.43% | +0.63% | +3.22% | 11.32% | 20.77 | 1.17 |
| Natural Gas | Bollinger | 145 | 37.24% | -1.91% | -3.00% | 11.15% | 23.98 | 0.63 |

Trade counts by commodity: Gold 510, Silver 513, Copper 487, Crude Oil 470, Natural Gas 519 (total 2,499). By strategy: MACD 793, Mean Reversion 980, Bollinger 726.

### 9.2 Expectancy matrix (average trade return, %)

| | MACD | Mean Reversion | Bollinger |
| --- | ---: | ---: | ---: |
| Gold | +0.73 | +0.13 | -0.03 |
| Silver | +1.83 | -0.17 | -0.05 |
| Copper | +0.85 | -0.33 | +1.15 |
| Crude Oil | +1.01 | -0.57 | +0.12 |
| Natural Gas | -2.27 | +0.63 | -1.91 |

### 9.3 Profit Factor matrix

| | MACD | Mean Reversion | Bollinger |
| --- | ---: | ---: | ---: |
| Gold | 1.38 | 1.10 | 0.98 |
| Silver | 1.62 | 0.94 | 0.98 |
| Copper | 1.34 | 0.86 | 1.58 |
| Crude Oil | 1.27 | 0.81 | 1.04 |
| Natural Gas | 0.64 | 1.17 | 0.63 |

### 9.4 Win rate matrix (%) and average holding period (days)

| | Win rate MACD | Win rate MR | Win rate BB | Hold MACD | Hold MR | Hold BB |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Gold | 40.9 | 65.5 | 38.6 | 44.2 | 22.5 | 24.9 |
| Silver | 37.4 | 62.0 | 34.6 | 43.5 | 22.6 | 25.3 |
| Copper | 41.0 | 67.0 | 39.6 | 42.8 | 21.4 | 26.8 |
| Crude Oil | 39.2 | 60.8 | 41.1 | 45.5 | 23.0 | 25.3 |
| Natural Gas | 29.4 | 64.4 | 37.2 | 34.8 | 20.8 | 24.0 |

### 9.5 Reading the matrix

**Observed.**

- Strategy performance is clearly **commodity-dependent within the tested specification**. Natural Gas reverses the pattern seen in the other four markets: MACD -2.27%, Mean Reversion +0.63%, Bollinger -1.91%.
- Eight of the 15 cells have positive expectancy and seven are negative. Five cells are within +/-0.20 percentage points of zero (Gold Mean Reversion, Gold Bollinger, Silver Mean Reversion, Silver Bollinger, Crude Oil Bollinger).
- MACD is positive in four of five commodities (Silver +1.83%, Crude Oil +1.01%, Copper +0.85%, Gold +0.73%) and negative in Natural Gas (-2.27%).
- Copper is the only commodity with positive expectancy under both MACD (+0.85%) and Bollinger (+1.15%), while Mean Reversion is -0.33%.
- Mean Reversion has the highest win rate in every commodity (60.77% to 67.01%) but positive expectancy in only two (Natural Gas +0.63%, Gold +0.13%).
- Average holding periods are stable within an archetype: MACD 34.83 to 45.49 days, Bollinger 23.98 to 26.77 days, Mean Reversion 20.77 to 23.04 days. Natural Gas MACD has the shortest MACD holding period (34.83 days versus 42.85 to 45.49 elsewhere).

**Statistical status.** None of the 15 cells shows a positive mean that survives Benjamini-Hochberg correction (Section 16).

---

## 10. Results: Strategy Archetype Comparison

| Metric | MACD | Mean Reversion | Bollinger |
| --- | ---: | ---: | ---: |
| Trades | 793 | 980 | 726 |
| Win rate | 37.33% | 63.98% | 38.15% |
| Mean | +0.341% | -0.056% | -0.163% |
| Median | -2.275% | +1.474% | -1.339% |
| Payoff ratio | 1.839 | 0.551 | 1.527 |
| Profit Factor | 1.095 | 0.979 | 0.944 |
| Std. Dev. | 12.266% | 8.419% | 8.785% |
| Average holding | 41.92 days | 22.06 days | 25.21 days |
| Skewness | +3.16 | -2.79 | +2.05 |
| Excess kurtosis | 19.1 | 14.0 | 9.9 |

Derived diagnostics (arithmetic on the reported values above):

| Diagnostic | MACD | Mean Reversion | Bollinger |
| --- | ---: | ---: | ---: |
| Break-even win rate, 1 / (1 + payoff) | 35.22% | 64.47% | 39.57% |
| Observed win rate minus break-even | +2.11 pp | -0.49 pp | -1.42 pp |
| Mean minus median | +2.62 pp | -1.53 pp | +1.18 pp |

### 10.1 Why a strategy can win 64% of trades and still have negative expectancy

Mean Reversion generated 980 trades: 627 winners and 353 losers.

| Quantity | Value |
| --- | ---: |
| Gross profit | +2,529.29% (sum of trade returns) |
| Gross loss | -2,584.00% |
| Profit Factor | 0.9788 |
| Average win (2,529.29 / 627) | about +4.03% |
| Average loss (2,584.00 / 353) | about 7.32% |
| Payoff ratio | 0.551 |
| Break-even win rate | 64.47% |
| Observed win rate | 63.98% |

The average loser is about 1.8 times the size of the average winner. To break even at that payoff the strategy would need to win 64.47% of its trades; it wins 63.98%. Expectancy is therefore slightly negative (-0.0558%), and it sits within half a percentage point of break-even, so small changes in win rate or loss size move its sign. Other pooled statistics: median +1.4737%, standard deviation 8.4188%, skewness -2.7873, excess kurtosis 14.0354, minimum -60.70%, maximum +25.16%.

### 10.2 Different return-generation mechanisms

| | Trend Following | Mean Reversion | Breakout |
| --- | --- | --- | --- |
| Typical trade | Small loss (median -2.28%) | Small gain (median +1.47%) | Small loss (median -1.34%) |
| What drives the mean | A minority of large winners | Offset by a minority of large losers | Minority of large winners |
| Distribution shape | Positive skew, heavy right tail | Negative skew, heavy left tail | Positive skew |
| Payoff | Above 1 (1.839) | Below 1 (0.551) | Above 1 (1.527) |
| Relation to break-even win rate | Above (+2.11 pp) | Slightly below (-0.49 pp) | Below (-1.42 pp) |

---

## 11. Results: Distributions and Mean vs Median

![Trade-return distributions](figures/03_trade_return_distributions.png)

*Figure 3. Empirical trade-return distributions by archetype. Display range is the pooled 1st to 99th percentile; 40 of 2,499 trades (1.6%) fall outside the display window and are retained in all statistics.*

| Archetype | Mean | Median | Skewness | Excess kurtosis |
| --- | ---: | ---: | ---: | ---: |
| MACD | +0.34% | -2.27% | +3.16 | 19.1 |
| Mean Reversion | -0.06% | +1.47% | -2.79 | 14.0 |
| Bollinger | -0.16% | -1.34% | +2.05 | 9.9 |

### 11.1 What these moments mean

- **Positive skew** (MACD, Bollinger): the long tail is on the right. Most trades are small losses; a few are large gains. The mean lies above the median.
- **Negative skew** (Mean Reversion): the long tail is on the left. Most trades are modest gains; a few are large losses. The mean lies below the median.
- **Excess kurtosis** far above zero (9.9 to 19.1) indicates tails much thicker than a normal distribution's. Extreme trades are far more common than a bell curve implies.
- **Why normality assumptions are weak here.** Using the pooled statistics, the worst Mean Reversion trade (-60.70%) lies about 7.2 standard deviations below that strategy's mean, and the best MACD trade in the archive (+117.7%) lies about 9.6 standard deviations above the MACD mean. Under a normal model such events have probabilities of order 10^-13 or smaller. Their presence is direct evidence of fat tails.
- **Why mean and standard deviation alone are insufficient.** Two strategies can share a mean and standard deviation and yet differ in the location of their risk: for one it sits in rare large gains, for the other in rare large losses. Skewness, kurtosis, the median and tail concentration are needed to describe that difference.

### 11.2 Mean vs median

Silver MACD: **mean +1.83%, median -2.09%**. The typical trade lost money while the arithmetic average was strongly positive. The gap of about 3.9 percentage points is produced by positive skew and large right-tail winners: the top tail of winning trades produces 50.6% of Silver MACD's gross profit (Section 13.1), and four of the ten largest trades in the entire archive are Silver MACD trades (Section 13.3). This is the classic trend-following distribution, and it means the mean and median must always be read together.

---

## 12. Results: Long vs Short Asymmetry

![Long vs short asymmetry](figures/04_long_short_asymmetry.png)

*Figure 4. Average return per long trade, per short trade, and their difference (long minus short). Positive differences indicate a long-side edge.*

### 12.1 Average return per trade by direction

| Commodity | Long MACD | Long MR | Long BB | Short MACD | Short MR | Short BB |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Gold | +1.97% | +0.97% | +0.62% | -0.89% | -0.55% | -1.43% |
| Silver | +3.18% | +0.84% | +0.42% | -0.42% | -0.91% | -0.84% |
| Copper | +1.49% | +0.26% | +1.46% | -0.45% | -0.84% | +0.68% |
| Crude Oil | +2.92% | -0.66% | +0.04% | -0.85% | -0.50% | +0.23% |
| Natural Gas | +0.17% | +1.92% | -0.48% | -3.34% | -0.29% | -3.52% |

### 12.2 Directional asymmetry (long expectancy minus short expectancy, percentage points)

| Commodity | MACD | Mean Reversion | Bollinger |
| --- | ---: | ---: | ---: |
| Gold | +2.86 | +1.53 | +2.06 |
| Silver | +3.59 | +1.75 | +1.26 |
| Copper | +1.93 | +1.11 | +0.78 |
| Crude Oil | +3.77 | -0.17 | -0.18 |
| Natural Gas | +3.51 | +2.21 | +3.04 |

(Differences are shown as annotated in Figure 4; small rounding differences versus the table above are possible.)

### 12.3 Bollinger, overall

| Direction | Trades | Win rate | Expectancy | Profit Factor |
| --- | ---: | ---: | ---: | ---: |
| Long | 440 | 41.59% | +0.43% | 1.17 |
| Short | 286 | 32.87% | -1.08% | 0.68 |

### 12.4 Observations and cautions

**Observed.**

- The long-minus-short difference is positive in 13 of 15 cells; the two exceptions are Crude Oil Mean Reversion (-0.17) and Crude Oil Bollinger (-0.18), both near zero.
- Long trades have positive average return in 13 of 15 cells; short trades have positive average return in only two (Copper Bollinger +0.68%, Crude Oil Bollinger +0.23%).
- Named examples: Silver MACD long +3.18% vs short -0.42%; Gold MACD long +1.97% vs short -0.89%; Natural Gas Mean Reversion long +1.92% vs short -0.29%; Natural Gas MACD long +0.17% vs short -3.34%.

**Statistical status.** There were **82 directional comparisons**. After FDR correction, **0** were statistically significant at the selected threshold. Even large long/short differences are therefore empirical asymmetries, not statistically proven effects.

**Interpretation (untested, no causal claim).** Because each direction contains roughly half of a cell's 134 to 205 trades, directional averages are noisier than cell averages. The sample also contained large upward moves in Gold, Silver and Copper (Figure 1), so a long-side advantage may partly reflect sample drift rather than anything structural about a strategy. This project does not test that explanation.

---

## 13. Results: Tail Concentration, Trimming and Extreme Trades

### 13.1 Profit and loss concentration

![Tail risk and profit concentration](figures/07_tail_risk_profit_concentration.png)

*Figure 7. Share of gross profits generated by the top tail of winning trades (top panel) and share of gross losses generated by the bottom tail of losing trades (bottom panel). The chart is descriptive and does not imply statistical significance.*

The chart's own labels define the buckets as the **top 10% of winning trades** and the **bottom 10% of losing trades**. Values below are read from the annotated bars.

| Commodity | Strategy | Top-tail share of gross profit | Bottom-tail share of gross loss |
| --- | --- | ---: | ---: |
| Gold | MACD | 36.2% | 27.0% |
| Gold | Mean Reversion | 27.2% | 33.4% |
| Gold | Bollinger | 31.9% | 31.8% |
| Silver | MACD | 50.6% | 24.9% |
| Silver | Mean Reversion | 24.3% | 45.4% |
| Silver | Bollinger | 46.7% | 23.8% |
| Copper | MACD | 47.6% | 26.1% |
| Copper | Mean Reversion | 26.5% | 42.4% |
| Copper | Bollinger | 39.5% | 27.4% |
| Crude Oil | MACD | 37.1% | 23.8% |
| Crude Oil | Mean Reversion | 23.3% | 45.8% |
| Crude Oil | Bollinger | 51.0% | 23.5% |
| Natural Gas | MACD | 38.9% | 25.1% |
| Natural Gas | Mean Reversion | 27.3% | 36.1% |
| Natural Gas | Bollinger | 36.7% | 29.5% |

Unweighted mean across the five commodities (arithmetic on the values above):

| Archetype | Top-tail share of gross profit | Bottom-tail share of gross loss |
| --- | ---: | ---: |
| MACD | 42.1% | 25.4% |
| Mean Reversion | 25.7% | 40.6% |
| Bollinger | 41.2% | 27.2% |

**Observed.** The concentration structure mirrors the distribution shapes. MACD and Bollinger profits are concentrated in a small number of winners (roughly 32% to 51% of gross profit in most cells), while their losses are spread more evenly (roughly 24% to 32%). Mean Reversion shows the reverse: profits are spread thinly (23% to 27%) but losses are concentrated in a few large trades (33% to 46%). Named examples: Silver MACD 50.6% of gross profit from the top tail and 24.9% of gross loss from the bottom tail; Crude Oil Bollinger 51.0% and 23.5%; Natural Gas Mean Reversion 27.3% and 36.1%.

**Why concentration matters for robustness.** A strategy whose profitability depends on a handful of trades has different statistical properties from one whose profitability is spread across many. Its sample mean is more sensitive to whether those few trades occurred within the sample, its confidence interval is wider, and it is psychologically harder to hold through the long stretches of small losses between large winners.

### 13.2 Trimming diagnostics

Trimming is used **only as a diagnostic** to understand tail dependence. Primary returns are never replaced by trimmed or winsorised values.

| Cell | Full mean | 5% trimmed mean | 10% trimmed mean |
| --- | ---: | ---: | ---: |
| Gold MACD | +0.73% | +0.14% | -0.21% |
| Gold Mean Reversion | +0.13% | +0.47% | +0.67% |
| Natural Gas Mean Reversion | +0.63% | +1.46% | +1.87% |

**Reading the direction of the change.**

- Gold MACD's mean shrinks as the tail is removed and turns negative at 10% trimming. Its positive expectancy depends on its right tail.
- Gold and Natural Gas Mean Reversion move the other way: removing extremes raises the mean. Their negative skew means the left tail pulls the average below what the typical trade earns.

### 13.3 Extreme trade analysis

![Extreme trade analysis](figures/08_extreme_trade_analysis.png)

*Figure 8. The ten worst and ten best individual trades in the frozen dataset. Extreme observations are retained in full and are not winsorised or capped. Entry and exit dates for each trade are printed in the figure.*

**Ten worst trades (all Mean Reversion):**

| Rank | Commodity | Position | Return |
| ---: | --- | --- | ---: |
| 1 | Natural Gas | Short | -60.70% |
| 2 | Silver | Short | -56.79% |
| 3 | Copper | Long | -53.71% |
| 4 | Copper | Short | -53.46% |
| 5 | Silver | Short | -53.03% |
| 6 | Crude Oil | Long | -46.99% |
| 7 | Silver | Short | -46.04% |
| 8 | Crude Oil | Long | -45.43% |
| 9 | Crude Oil | Long | -43.88% |
| 10 | Natural Gas | Short | -42.60% |

**Ten best trades (values read from Figure 8):**

| Rank | Commodity | Strategy | Return |
| ---: | --- | --- | ---: |
| 1 | Silver | MACD | +117.7% |
| 2 | Natural Gas | MACD | +91.4% |
| 3 | Silver | MACD | +71.9% |
| 4 | Silver | MACD | +70.6% |
| 5 | Copper | Bollinger | +53.5% |
| 6 | Silver | MACD | +53.1% |
| 7 | Silver | Bollinger | +53.0% |
| 8 | Copper | MACD | +52.8% |
| 9 | Crude Oil | MACD | +51.1% |
| 10 | Natural Gas | MACD | +50.9% |

**Observed.**

- All ten worst trades are Mean Reversion trades. Of the ten best, eight are MACD and two are Bollinger; none is Mean Reversion.
- The extremes span four commodities; Gold appears in neither list.
- Silver contributes five of the ten best trades and three of the ten worst (worst-list ranks 2, 5 and 7).
- Mean Reversion's worst trade (-60.70%, Natural Gas short) is much larger in magnitude than its best (+25.16%).

**Validation.** Every extreme trade was checked for entry price, exit price, entry date, exit date, direction, holding period, calculated return and underlying price path. The result was **0 validation flags**, so all extreme winners and losers were retained.

**Economic reason for the severe Mean Reversion left tail.** A market can remain displaced from its historical mean for much longer than a mean-reversion trade can remain economically attractive. Because the specified exit is a return of Z through zero, a position in a market that keeps moving away from its recent average stays open and accumulates losses (Section 21.2).

---

## 14. Results: Temporal Stability

![Temporal stability](figures/05_temporal_stability.png)

*Figure 5. Average trade return by fixed historical period for each commodity, shown separately for each strategy. Persistent positioning above or below zero indicates temporal consistency; large swings indicate regime sensitivity.*

The full sample is divided into five fixed periods: **2000-2005, 2006-2010, 2011-2015, 2016-2020, 2021-2026** (the first window begins at the sample start of 2000-08-30 and the last ends at 2026-09-04). Each commodity x strategy combination is evaluated in every period to ask whether a relationship is persistent, episodic, regime-dependent or unstable.

**Observed.**

- **Silver MACD remained positive across all five historical periods examined.**
- Several other combinations show considerably more variation through time, including sign changes between periods. The widest swings in Figure 5 appear in the Natural Gas lines.
- Exact per-period values are stored in the temporal statistics outputs in the research archive.

**Caution.** With 2,499 trades spread over 15 cells and 5 periods, an even split would leave only about 33 trades per commodity x strategy x period cell. Period averages are therefore noisy, and persistence across five noisy averages is descriptive evidence, not proof of future performance.

---

## 15. Results: Market-State (Regime) Analysis

### 15.1 Causal feature construction

| Feature | Description |
| --- | --- |
| 20-day momentum | Price change over 20 trading days |
| 60-day momentum | Price change over 60 trading days |
| 20-day realised volatility | Annualised realised volatility over 20 days |
| 60-day realised volatility | Annualised realised volatility over 60 days |
| SMA200 | 200-day simple moving average |
| Distance from SMA200 | Percentage distance of price from SMA200 |
| 20-day positive-day fraction | Share of the last 20 days with positive returns |

**All regime inputs are shifted by one observation**, so classifying a trade never uses information from the current trading day. This is what keeps the regime analysis free of look-ahead bias.

### 15.2 State definitions

| Dimension | State | Rule |
| --- | --- | --- |
| Trend | UPTREND | Distance from SMA200 > +5% |
| Trend | DOWNTREND | Distance from SMA200 < -5% |
| Trend | SIDEWAYS | Between -5% and +5% |
| 20-day momentum | POSITIVE_MOMENTUM | > +5% |
| 20-day momentum | NEGATIVE_MOMENTUM | < -5% |
| 20-day momentum | NEUTRAL | Between -5% and +5% |
| Volatility | HIGH | 20-day annualised volatility > 125% of its expanding median |
| Volatility | LOW | < 75% of its expanding median |
| Volatility | NORMAL | Between the two thresholds |

The volatility comparison uses a **causal expanding median** (only history up to that date). Because the states are defined as a ratio to the market's own median, the annualisation constant cancels out and each market is compared with its own history.

### 15.3 Seven combined states

`UPTREND_HIGH_VOL`, `UPTREND_NORMAL_LOW_VOL`, `DOWNTREND_HIGH_VOL`, `DOWNTREND_NORMAL_LOW_VOL`, `SIDEWAYS_HIGH_VOL`, `SIDEWAYS_LOW_VOL`, `SIDEWAYS_NORMAL_VOL`

Uptrend and downtrend states combine normal and low volatility, while sideways separates all three volatility levels. Overall regime-state coverage was approximately **97.68%**.

### 15.4 Conditional patterns observed

| Cell | Market state | Approx. mean | Approx. PF | Approx. win rate |
| --- | --- | ---: | ---: | ---: |
| Gold MACD | Uptrend + normal/low volatility | +2.84% | 2.64 | n/a |
| Gold MACD | Downtrend | -2.71% | n/a | n/a |
| Gold Mean Reversion | Downtrend | +1.15% | n/a | 73.68% |
| Copper MACD | Uptrend + high volatility | +5.83% | 3.93 | n/a |
| Copper Bollinger | Uptrend + normal/low volatility | +1.81% | 2.16 | n/a |
| Natural Gas Mean Reversion | Uptrend + high volatility | +5.00% | 3.41 | n/a |
| Natural Gas Mean Reversion | Downtrend + normal/low volatility | +1.61% | 1.61 | n/a |

These are described as **economically interesting conditional patterns, not proven causal regime effects.**

### 15.5 Statistical qualification

The regime study covered **2,499 regime trades** and **240 conditional observations/tests** across the broader regime framework. After multiple-testing correction with the False Discovery Rate procedure:

> **0 commodity x strategy best-vs-worst market-state comparisons were statistically significant at the 5% FDR level.**

The regime analysis therefore generates hypotheses. The available evidence does **not** establish statistically robust regime effects.

Why the conditional cells should be treated cautiously:

- **Small cells.** Each commodity x strategy cell holds 134 to 205 trades. Splitting them across seven states leaves many state cells with few trades, so extreme Profit Factors such as 3.93 or 3.41 rest on small samples.
- **Selection effect.** Comparing the best state with the worst state, chosen after the fact, mechanically overstates differences (the range of many noisy averages is wider than the difference between two pre-specified ones).
- **Many looks.** 240 tests give many opportunities for chance patterns. This is precisely what FDR correction guards against.

---

## 16. Statistical Validation

### 16.1 Procedures applied to the 15 core cells

| Output | Description |
| --- | --- |
| Mean, standard deviation | Trade-level moments per cell |
| t-statistic and p-value | Test of the mean trade return against zero |
| Bootstrap confidence interval | 95% interval for the mean trade return |
| Probability bootstrap mean > 0 | Share of bootstrap means above zero |
| Adjusted p-value | Benjamini-Hochberg FDR-adjusted |

**Bootstrap settings: 10,000 iterations, random seed 42.** Trade returns are resampled to build an empirical sampling distribution of the mean. This suits trade returns because they are strongly non-normal (skew -2.79 to +3.16, excess kurtosis 9.9 to 19.1), so classical normal-theory intervals rest on assumptions the data visibly violate. The bootstrap treats trades as exchangeable units, so it does not model serial dependence between trades (Section 23).

**Benjamini-Hochberg procedure.** With m tests and target rate q = 5%, sort the p-values p(1) <= ... <= p(m), find the largest k such that

```math
p_{(k)} \le \frac{k}{m}\,q
```

and reject hypotheses 1 through k. The adjusted p-value is p_adj(i) = min over j >= i of (m / j) * p(j). This controls the expected share of false discoveries among the results declared significant.

**Why correction matters.** If 15 independent tests were each run at a 5% level with no true effects, the chance of at least one false positive would be about 1 - 0.95^15, roughly 54%. Correction is necessary before treating any single positive cell as a discovery.

### 16.2 Core statistical result

Across the 15 commodity x strategy combinations:

- **13 were inconclusive.**
- **2 produced negative bootstrap evidence:**

| Cell | Mean | Bootstrap 95% CI | Probability bootstrap mean > 0 |
| --- | ---: | --- | ---: |
| Natural Gas MACD | -2.27% | about [-4.30%, -0.13%] | about 2.01% |
| Natural Gas Bollinger | -1.91% | about [-3.63%, -0.07%] | about 2.14% |

- **No positive commodity x strategy combination survived Benjamini-Hochberg FDR correction at the 5% level.**

Silver MACD had the strongest **observed** historical expectancy among the tested combinations. It is **not** statistically proven alpha.

### 16.3 Why Silver MACD is not statistically established (illustrative arithmetic)

The table below applies the simple approximation SE = SD / sqrt(N) and t = mean / SE to the reported summary statistics, treating trades as independent. These figures are illustrative arithmetic on values already shown in Section 9, not the archive's own test outputs; the archive's t-statistics, p-values and bootstrap intervals are the authoritative results.

| Cell | Mean | SD | N | SE = SD / sqrt(N) | Mean / SE |
| --- | ---: | ---: | ---: | ---: | ---: |
| Silver MACD | +1.83% | 15.23% | 155 | 1.22 | +1.50 |
| Copper Bollinger | +1.15% | 8.41% | 134 | 0.73 | +1.58 |
| Copper MACD | +0.85% | 9.65% | 156 | 0.77 | +1.10 |
| Gold MACD | +0.73% | 6.75% | 154 | 0.54 | +1.34 |
| Crude Oil MACD | +1.01% | 12.59% | 148 | 1.03 | +0.98 |
| Natural Gas Mean Reversion | +0.63% | 11.32% | 194 | 0.81 | +0.78 |
| Natural Gas MACD | -2.27% | 14.38% | 180 | 1.07 | -2.12 |
| Natural Gas Bollinger | -1.91% | 11.15% | 145 | 0.93 | -2.06 |

The positive cells sit roughly 1 to 1.6 standard errors from zero. Silver MACD has the largest mean but also the largest trade dispersion, so its signal-to-noise ratio is not higher than several smaller-mean cells. At the same mean and standard deviation, about 266 trades (versus 155 observed) would be needed to reach |t| of 1.96 even before any multiple-testing correction. The two Natural Gas cells are the only ones whose means lie about two standard errors on the negative side, consistent with their bootstrap intervals excluding zero.

### 16.4 Directional testing

There were **82 directional comparisons**. After FDR correction, **0** were statistically significant at the selected threshold.

### 16.5 Tail robustness of conditional results

Direction of the observed conditional relationships was preserved in:

| Trimming level | Share of comparisons with preserved direction |
| --- | ---: |
| 5% trimming | 80.70% |
| 10% trimming | 70.18% |

Most observed relationships are therefore not driven entirely by individual extreme trades. The decline at 10% trimming shows meaningful tail sensitivity.

---

## 17. Primary Results vs Robustness Diagnostics

| Primary results | Robustness diagnostics |
| --- | --- |
| Actual observed trade-level returns | Trimmed means at 5% and 10% |
| Retained extreme winners and losers | Direction-preservation checks under trimming |
| Frozen trade archive of 2,499 trades | Extreme-trade path validation |
| Not winsorised | Bootstrap and FDR analysis |
| Not capped | Temporal and regime splits |

- **No winsorisation of primary results.** Trimming and winsorising are used only to understand tail dependence. They do not replace the original trade returns.
- **No return capping.** The research archive and visualization dataset do not substitute capped values for observed trade returns.
- **Frozen dataset.** The visualization stage does not rerun strategies, download new data, optimise parameters, alter trades, winsorise primary returns, or cap returns.

---

## 18. Descriptive Price Behaviour

![Cumulative price returns](figures/01_cumulative_price_returns.png)

*Figure 1. Cumulative percentage change from each commodity's first valid positive closing price.*

This chart shows **descriptive price behaviour only**. It is **not a strategy equity curve** and says nothing about strategy performance.

What it shows:

- Gold, Silver and Copper show substantial cumulative price appreciation across the sample, with Silver's path the most volatile, including a sharp peak and pullback in 2026.
- Crude Oil is comparatively flat over the full period and falls below -100% around April 2020 because the price itself turned negative (Section 5.6).
- Natural Gas finishes the sample below its starting level.

These price paths give context for the long/short asymmetry results: the sample contains strong upward drift in several markets, which is a plausible contributor to long-side advantages (Section 12.4), although this project does not test that link.

---

## 19. Final Research Dashboard

The dashboard (shown at the top of this README) summarises 2,499 frozen trades across 5 commodities and 3 archetypes for 2000-2026, with four heatmaps (expectancy, Profit Factor, win rate, average holding period) and a key-findings panel.

Findings stated on the dashboard:

- Strongest observed expectancy: Silver - MACD Trend (+1.83%).
- Weakest observed expectancy: Natural Gas - MACD Trend (-2.27%).
- Mean Reversion has the highest average win rate across the five commodities (64.0%), but win rate alone does not determine profitability.
- Observed performance varies materially across commodities and strategy archetypes and should be considered alongside distributional, temporal and regime evidence.

The dashboard footer states that no strategy rerun, optimisation, winsorisation or return capping was performed for the visualization.

---

## 20. Key Findings

Each finding is labelled with its evidence status.

**1. Strategy performance varies materially by commodity.**
MACD expectancy: Silver +1.83%, Crude Oil +1.01%, Copper +0.85%, Gold +0.73%, Natural Gas -2.27%.
*Status: Observed. No positive cell survived FDR.*

**2. Win rate does not equal profitability.**
Mean Reversion: 63.98% win rate, -0.0558% mean, Profit Factor about 0.98. Its win rate sits about 0.5 percentage points below the 64.47% break-even level implied by its payoff.
*Status: Observed.*

**3. Trend following and mean reversion have opposite distribution shapes.**
MACD: low win rate, positive mean, positive skew, large right-tail winners. Mean Reversion: high win rate, near-zero or negative mean, negative skew, large left-tail losses.
*Status: Observed.*

**4. Direction matters descriptively.**
Silver MACD long +3.18% vs short -0.42%; Gold MACD long +1.97% vs short -0.89%; Natural Gas Mean Reversion long +1.92% vs short -0.29%.
*Status: Observed. 0 of 82 directional comparisons significant after FDR. No causal claim.*

**5. Natural Gas behaves differently from the other commodities.**
It has the highest daily volatility (3.86%), yet MACD -2.27%, Mean Reversion +0.63% and Bollinger -1.91%.
*Status: Observed. Not evidence that Natural Gas is inherently mean-reverting.*

**6. Silver MACD is the strongest observed historical combination.**
+1.83% expectancy, PF 1.62, 37.42% win rate, 43.53-day average holding, but a -2.09% median: its average is influenced materially by positive skew and large winners.
*Status: Observed. Not statistically proven.*

**7. Copper is positive under both trend and breakout logic.**
MACD +0.85% and Bollinger +1.15%, while Mean Reversion is -0.33%.
*Status: Observed.*

**8. Mean Reversion has a significant negative tail.**
Worst observed trade: Natural Gas short at -60.70%. All ten worst trades in the archive are Mean Reversion trades.
*Status: Observed; extreme trades validated (0 flags).*

**9. Bollinger behaviour depends on commodity and direction.**
Overall long +0.43% (PF 1.17) versus short -1.08% (PF 0.68).
*Status: Observed. No causal claim.*

**10. Temporal stability matters.**
Silver MACD remained positive across the five historical periods examined; several other cells varied considerably.
*Status: Observed (descriptive, roughly 33 trades per cell-period).*

**11. Regime analysis produces interesting conditional patterns but no significant positive state effects.**
No best-vs-worst market-state comparison survived 5% FDR correction.
*Status: Tested. Hypothesis-generating only.*

---

## 21. Interpretation: Why the Results May Look the Way They Do

*This section is **Interpretation**: explanations offered to make the numbers understandable. None of these mechanisms is tested by this project.*

### 21.1 Trend following (MACD)

The exit rule cuts losses at a 20-day channel break but has no profit target, so winners run until the trend reverses enough to break the opposite channel. This produces frequent small losses and occasional large gains: a low win rate (37.33%), a payoff above 1 (1.839), positive skew (+3.16) and a mean above the median. Because expectancy lives in the right tail, it is fragile to whether a few large trends fall inside the sample; Silver MACD's top-tail winners make up 50.6% of its gross profit, and the four largest Silver MACD winners printed in Figure 8 occur in 2005-06, 2010-11, 2020 and 2025-26, periods of sharp price rises visible in Figure 1. Commonly proposed reasons for trend persistence in commodities (slow adjustment of supply and demand, inventory dynamics, investor behaviour) were not tested here.

### 21.2 Mean reversion (Hurst + Z-score)

The payoff structure is the key. Gains are structurally bounded, because the trade exits when Z returns through zero, so a winner earns roughly the distance from a 2-standard-deviation displacement back to the mean (average win about 4.03%). Losses are open-ended, because a market that keeps moving away from its recent average keeps the position open (average loss about 7.32%, worst trade -60.70%). The high win rate (63.98%) is therefore a necessary compensation for a payoff ratio of 0.551, and the break-even requirement (64.47%) leaves almost no margin. The negative skew (-2.79) and loss concentration (33% to 46% of gross loss from the bottom tail) follow directly.

A noisy rolling Hurst estimate (Section 7.3) means the H < 0.5 filter is a weak screen for genuinely anti-persistent conditions. The positive Natural Gas result (+0.63%, median +3.22%, win rate 64.43%) has one candidate explanation not tested here: sharp Natural Gas price spikes have often been followed by partial reversals, a pattern commonly discussed alongside storage and weather-driven demand. The project does not conclude that Natural Gas is inherently mean reverting.

### 21.3 Breakout (Bollinger)

Breakout entries buy or sell at the moment volatility expands, and the short 10-day channel exit ends trades quickly when momentum stalls. The result is a low win rate (38.15%) with a payoff above 1 (1.527), but a break-even win rate of 39.57%, slightly above what was observed, giving a small negative overall expectancy (-0.163%). The pronounced long/short gap (long +0.43% versus short -1.08%) is consistent with the upward drift in several markets (Figure 1), an explanation that is not tested here.

### 21.4 Cross-cutting: volatility and identical parameters

The same indicator settings are applied to markets whose daily volatility differs by a factor of about 3.4 (Gold versus Natural Gas). In a market with very frequent large reversals, a fixed 12/26/9 crossover and a fixed 20-day channel produce more false signals and earlier stop-outs, which is consistent with Natural Gas MACD having the lowest win rate (29.44%) and the shortest MACD holding period (34.83 days). This is a hypothesis about interaction between strategy mechanics and distribution, which is the central question of the project, not a tested result.

---

## 22. What This Project Does Not Claim

This project does **not** claim:

- guaranteed profitability;
- future performance of any kind;
- statistically proven alpha;
- that Silver MACD will continue to work;
- that Natural Gas is inherently mean reverting;
- that regime classifications cause returns;
- complete net-of-cost portfolio returns;
- exact historical P&L from individual futures contracts;
- fully tradable continuous-contract performance;
- portfolio-level Sharpe, Sortino or Treynor results.

Sharpe, Sortino and Treynor ratios are intentionally not reported, because they were not validly established in this research.

---

## 23. Limitations

| # | Limitation | Why it matters |
| --- | --- | --- |
| 1 | **Continuous futures construction.** Results use supplied continuous series. | Roll discontinuities can appear as returns, and the series is not the P&L of a specific contract. |
| 2 | **No full transaction-cost model.** No commissions, bid/ask spread, market impact, or liquidity model. | Headline expectancy and Profit Factor are gross. Small expectancies (for example +0.13%) could be eliminated by realistic costs. |
| 3 | **No custom contract-roll reconstruction.** | Roll yield, contango and backwardation are part of true futures returns and are not modelled. |
| 4 | **No portfolio construction.** No sizing, leverage, margin, or overlapping-trade treatment. | Trade-level statistics do not translate into portfolio returns or portfolio risk. |
| 5 | **Trade-level rather than portfolio-level returns.** | A +1.83% trade expectancy is not a +1.83% periodic return. |
| 6 | **Historical non-stationarity.** Volatility, correlations and market structure changed over 26 years. | Period averages differ across time, and past behaviour need not persist. |
| 7 | **Possible dependence between trades.** Trades overlap in time across markets and share macro drivers. | Independence assumptions in t-tests and trade-level bootstrap can overstate precision. |
| 8 | **Multiple-testing problem.** 15 cell tests, 82 directional comparisons, 240 conditional tests. | Some apparent patterns will arise by chance. FDR correction addresses this, and its result is that nothing positive survives. |
| 9 | **Limited trades per cell.** 134 to 205 trades per commodity x strategy cell. | With fat-tailed returns, precision is low; a few trades can move a cell's mean. |
| 10 | **No out-of-sample validation yet.** | All results are in-sample descriptions of one historical path. |
| 11 | **No walk-forward optimisation or validation.** | Parameter robustness across time has not been tested. |
| 12 | **No live execution validation.** | Slippage, gaps, order handling and data-feed behaviour are not observed. |

Additional data caveat: the Crude Oil April 2020 negative-price event is handled by skipping invalid execution prices (Section 5.6), which is not an economic treatment of trading negative prices.

---

## 24. Reproducibility and Archive

The project is designed so that analysis can resume from saved CSV outputs rather than rerunning the whole backtest.

### 24.1 Archive folders

| Folder | Contents |
| --- | --- |
| `trades/` | Per-strategy trade logs and the combined trade log |
| `statistics/` | Strategy and commodity statistics, directional analysis, expectancy / Profit Factor / win-rate / holding-period matrices, statistical significance, tail, regime and conditional analysis, economic decomposition |
| `distributions/` | Distribution outputs |
| `validation/` | Data audit and extreme-trade validation outputs, archive verification |
| `market_data/` | Raw daily market data (see licensing note in Section 27) |
| `metadata/` | Research manifests, visualization mapping |

The archive also contains the causal regime features.

### 24.2 Archive integrity checklist

| Check | Expected value |
| --- | --- |
| Combined trade log rows | 2,499 |
| Trades by strategy (MACD / Mean Reversion / Bollinger) | 793 / 980 / 726 |
| Trades by commodity (Gold / Silver / Copper / Crude Oil / Natural Gas) | 510 / 513 / 487 / 470 / 519 |
| Raw observations by commodity (Gold / Silver / Copper / Crude Oil / Natural Gas) | 6,528 / 6,530 / 6,533 / 6,537 / 6,534 (total 32,662) |
| Common sample | 2000-08-30 to 2026-09-04 |
| Duplicate observations / missing OHLCV | 0 / 0 |
| Bootstrap | 10,000 iterations, seed 42 |
| Extreme-trade validation flags | 0 |
| Mean Reversion gross profit / gross loss / PF | +2,529.29% / -2,584.00% / 0.9788 |

Template for a quick check (adjust the file path and column names to your CSV headers):

```python
import pandas as pd

trades = pd.read_csv("trades/combined_trade_log.csv")   # adjust to the actual file name
print(len(trades))                       # expect 2499
print(trades["Strategy"].value_counts()) # expect 793 / 980 / 726
print(trades["Ticker"].value_counts())   # expect 510 / 513 / 487 / 470 / 519
```

### 24.3 How to reproduce

1. Clone the repository and create a Python environment.
2. Install dependencies: `pip install -r requirements.txt`.
3. **Reproduce the figures only (fastest):** open the visualization notebook and run all cells. It reads from the frozen archive and does not rerun strategies.
4. **Reproduce the full pipeline:** run the research notebooks in order (data acquisition and validation, strategy engine and trade generation, then statistical analysis).
5. Compare outputs with the integrity checklist above.

**Important.** Re-downloading Yahoo Finance data may not reproduce the archive exactly, because the provider can revise or re-adjust continuous series. The frozen archive is the reference for all reported numbers. Bootstrap results depend on the fixed seed (42).

---

## 25. Repository Structure

```
commodity-futures-strategy-archetype-research/
|-- README.md
|-- LICENSE
|-- requirements.txt
|-- .gitignore
|
|-- notebooks/                 research pipeline notebooks and visualization notebook
|
|-- research_archive/          frozen research archive
|   |-- market_data/
|   |-- trades/
|   |-- statistics/
|   |-- distributions/
|   |-- validation/
|   +-- metadata/
|
+-- figures/                   exported chart images used in this README
    |-- 01_cumulative_price_returns.png
    |-- 02_commodity_strategy_performance.png
    |-- 03_trade_return_distributions.png
    |-- 04_long_short_asymmetry.png
    |-- 05_temporal_stability.png
    |-- 06_statistical_evidence_robustness.png
    |-- 07_tail_risk_profit_concentration.png
    |-- 08_extreme_trade_analysis.png
    +-- 09_final_research_dashboard.png
```

### Visualization framework

The dedicated visualization notebook produces the following sequence:

| # | Figure | Content |
| --- | --- | --- |
| 1 | Cumulative Commodity Price Returns | Descriptive price behaviour (not a strategy equity curve) |
| 2 | Commodity x Strategy Performance | Expectancy, Profit Factor, win rate, holding period |
| 3 | Trade-Return Distributions | Empirical distributions with skew and excess kurtosis |
| 4 | Long vs Short Asymmetry | Directional expectancy and asymmetry |
| 5 | Temporal Stability | Average trade return across five periods |
| 6 | Statistical Evidence and Robustness | Bootstrap and FDR evidence |
| 7 | Tail Risk and Profit Concentration | Concentration of gross profits and losses |
| 8 | Extreme Trade Analysis | Ten best and ten worst trades |
| 9 | Final Research Dashboard | Summary heatmaps and key findings |

<!-- Figure 6 (Statistical Evidence and Robustness): add here once exported, e.g.
![Statistical evidence and robustness](figures/06_statistical_evidence_robustness.png)
-->

---

## 26. Future Research

The following are **future research directions, not completed components** of this project:

- Futures-specific contract reconstruction and explicit roll methodology
- Contango / backwardation analysis
- Transaction-cost, slippage, liquidity and margin modelling
- Volatility targeting, risk parity and portfolio construction
- Walk-forward testing and out-of-sample validation
- Cross-sectional commodity strategies
- Carry, momentum, trend and volatility factors
- Regime-conditioned models and macro regime variables (inflation, USD, liquidity, commodity-shock regimes)

---

## 27. Data Licensing, Disclaimer and License

**Data.** Market data are sourced from Yahoo Finance and remain subject to the provider's terms. The MIT License covers the project's original code and original written material only. It does not grant rights to third-party market data.

**Disclaimer.** This repository is for research, educational and analytical purposes only. Nothing in it constitutes investment, financial, tax or other professional advice. Historical results are not indicative of future performance, and nothing here is a recommendation to buy, sell or hold any security, commodity or derivative.

**License.** MIT. See the `LICENSE` file.

---

**Keywords:** quantitative-finance, commodity-futures, systematic-trading, trend-following, mean-reversion, hurst-exponent, bollinger-bands, macd, backtesting, statistical-analysis, bootstrap, false-discovery-rate, regime-analysis, python, jupyter-notebook
