# Commodity-Equity Linkage Study: Research Documentation

## Analyzing the Relationship between Crude Oil Prices and Indian Transportation Stocks

---

## Table of Contents

1. [Research Overview](#1-research-overview)
2. [Data Description](#2-data-description)
3. [Methodology](#3-methodology)
4. [Exploratory Data Analysis](#4-exploratory-data-analysis)
5. [Correlation Analysis](#5-correlation-analysis)
6. [Statistical Testing](#6-statistical-testing)
7. [Regression Analysis](#7-regression-analysis)
8. [Event Study Analysis](#8-event-study-analysis)
9. [VAR Model & Impulse Response](#9-var-model--impulse-response)
10. [Volatility Regime Analysis](#10-volatility-regime-analysis)
11. [Sub-Period Analysis](#11-sub-period-analysis)
12. [Key Findings & Conclusions](#12-key-findings--conclusions)
13. [Limitations & Future Work](#13-limitations--future-work)
14. [Appendix: Output Files](#14-appendix-output-files)

---

## 1. Research Overview

### 1.1 Research Objective

This study investigates the nature and magnitude of the relationship between crude oil price fluctuations and the stock performance of Indian transportation and energy-related companies. The central question is: **Do changes in crude oil prices systematically affect the equity returns of Indian firms whose operations are heavily tied to fuel costs?**

The analysis is structured around several sub-questions:

- What is the direction and strength of correlation between crude oil and transportation/energy stocks?
- Does crude oil Granger-cause movements in these stocks (i.e., do oil prices lead stock returns)?
- Is the relationship symmetric across sectors (airlines vs. logistics vs. oil marketing companies)?
- Does the strength of the commodity-equity linkage vary with market volatility?
- Are there long-run equilibrium relationships (cointegration) between crude oil and stock prices?

### 1.2 Economic Rationale

Crude oil is a critical input cost for the transportation sector. Its price movements affect:

- **Airlines**: Aviation turbine fuel (ATF) constitutes 35-45% of an airline's operating cost. Rising crude prices compress margins unless passed through via fare hikes.
- **Logistics & Trucking**: Diesel fuel is a major cost component. Higher crude prices increase transportation costs.
- **Shipping**: Bunker fuel costs are directly linked to crude oil. International shipping companies face both cost and demand effects.
- **Oil Marketing Companies (OMCs)**: IOC, BPCL, and HINDPETRO operate in a regulated pricing environment. They benefit from inventory gains when crude rises (buying low, selling high from existing stock) but face margin pressure under price controls.

The theoretical expectation is:
- **Negative correlation** for airlines and logistics (crude oil is a cost; higher oil = lower profitability)
- **Mixed/Positive correlation** for OMCs (inventory gains, marketing margins, and government subsidy dynamics)

### 1.3 Analysis Period

| Parameter | Value |
|-----------|-------|
| **Start Date** | January 1, 2025 |
| **End Date** | February 10, 2026 |
| **Total Trading Days** | 291 |
| **Effective Data Points (after cleaning)** | ~130 daily observations for return analysis |
| **Frequency** | Daily |

This 13-month window captures a period characterized by global macroeconomic uncertainty, OPEC+ production decisions, geopolitical tensions, and evolving monetary policy across major economies.

---

## 2. Data Description

### 2.1 Data Sources

| Source | Data Type | Details |
|--------|-----------|---------|
| **Zerodha Kite API** | MCX Crude Oil Futures, NSE Stocks, Indices | Real-time Indian market data via authenticated API |
| **Yahoo Finance** | Brent Crude (BZ=F), WTI Crude (CL=F), USD/INR (USDINR=X) | International benchmarks and FX rates |
| **FRED API** | VIX (VIXCLS), US Dollar Index (DTWEXBGS), WTI (DCOILWTICO), Brent (DCOILBRENTEU) | Macroeconomic indicators |

### 2.2 Instruments Analyzed

#### Commodity
- **MCX Crude Oil Futures** (INR-denominated, continuous contract stitched from multiple expiry contracts)

#### Global Benchmarks
- **Brent Crude** (BZ=F) - ICE Brent continuous futures, USD/barrel
- **WTI Crude** (CL=F) - NYMEX WTI continuous futures, USD/barrel

#### Transportation & Energy Stocks (10 instruments)

| Stock | Symbol | Sub-Sector | Description |
|-------|--------|------------|-------------|
| IndiGo | INDIGO | Airlines | InterGlobe Aviation Ltd, India's largest airline |
| Blue Dart | BLUEDART | Logistics | Express logistics and supply chain |
| VRL Logistics | VRLLOG | Logistics | Road transportation and logistics |
| TCI Express | TCIEXP | Logistics | Express distribution services |
| Mahindra Logistics | MAHLOG | Logistics | Third-party logistics provider |
| Shipping Corp of India | SCI | Shipping | Government-owned shipping company |
| Great Eastern Shipping | GESHIP | Shipping | Private-sector shipping company |
| Indian Oil Corporation | IOC | Oil Marketing | India's largest OMC |
| Hindustan Petroleum | HINDPETRO | Oil Marketing | Major fuel retailer |
| Bharat Petroleum | BPCL | Oil Marketing | Downstream oil company |

#### Market Indices
- **NIFTY 50** - Broad market benchmark
- **NIFTY AUTO** - Sector benchmark for auto and transportation

#### Currency
- **USD/INR** - Exchange rate (critical for translating international crude prices to INR costs)

#### Macroeconomic Indicators
- **CBOE VIX** - Global risk sentiment proxy
- **US Dollar Index (DXY)** - Dollar strength indicator
- **WTI (FRED)** and **Brent (FRED)** - Alternative crude price sources for cross-validation

### 2.3 Master Dataset Structure

The final master dataset contains **291 rows x 20 columns**, with the following structure:

| Column | Description | Coverage |
|--------|-------------|----------|
| MCX_Crude | MCX Crude Oil futures close price (INR) | Partial (contract-based) |
| Brent_Crude | Brent crude futures close (USD) | Full |
| WTI_Crude | WTI crude futures close (USD) | Full |
| USDINR | USD/INR exchange rate | Full |
| Stock_INDIGO ... Stock_BPCL | Daily closing prices for 10 stocks | Full |
| Index_NIFTY50, Index_NIFTYAUTO | Index levels | Full |
| VIX, DXY, WTI_FRED, BRENT_FRED | Macro indicators | Partial |

### 2.4 Data Preprocessing

1. **Timezone Alignment**: All data sources were aligned to IST (Asia/Kolkata, UTC+05:30) to ensure proper date matching between domestic and international sources.
2. **Multi-level Column Handling**: Yahoo Finance returns multi-level column headers; these were flattened for uniform access.
3. **Missing Data Strategy**:
   - Columns with <20% data coverage were dropped.
   - Forward-fill (limit=5 days) for minor gaps (holidays, data misalignment).
   - Backward-fill (limit=5 days) for leading gaps.
   - Rows with >50% NaN values across all columns were dropped.
   - Rows where all crude oil columns were missing were removed.
4. **Continuous Crude Oil Series**: MCX crude futures were stitched across contract expiries by fetching data from each contract and keeping the most recent contract for overlapping dates.
5. **Return Calculation**: Log returns were computed as `ln(P_t / P_{t-1})` for all price series. Log returns are preferred for their additive property over time and better statistical properties (closer to normality).

---

## 3. Methodology

### 3.1 Analytical Framework

The study employs a multi-method approach to capture different dimensions of the commodity-equity relationship:

```
                    ┌──────────────────────────────────┐
                    │     Commodity-Equity Linkage      │
                    └──────────────────┬───────────────┘
                                       │
          ┌────────────────┬───────────┼───────────┬────────────────┐
          │                │           │           │                │
    ┌─────┴─────┐   ┌─────┴─────┐ ┌───┴───┐ ┌────┴─────┐   ┌─────┴─────┐
    │Correlation │   │Causality  │ │Regres-│ │Event     │   │Volatility │
    │Analysis    │   │Testing    │ │sion   │ │Study     │   │Analysis   │
    └─────┬─────┘   └─────┬─────┘ └───┬───┘ └────┬─────┘   └─────┬─────┘
          │                │           │           │                │
     Pearson &        Granger       OLS &      CAR around       High vs Low
     Spearman        Causality    Lagged OLS   Oil Shocks     Vol Regimes
     Rolling Corr    Cointegration  VAR/IRF
```

### 3.2 Statistical Methods

| Method | Purpose | Null Hypothesis |
|--------|---------|-----------------|
| **Pearson Correlation** | Measure linear association between crude oil and stock returns | No linear relationship (r = 0) |
| **Spearman Correlation** | Measure monotonic (rank-based) association, robust to outliers | No monotonic relationship (rho = 0) |
| **Rolling Correlation** | Capture time-varying nature of the relationship (90-day window) | N/A (descriptive) |
| **ADF Test** | Test stationarity of return series (prerequisite for causality/VAR) | Series has a unit root (non-stationary) |
| **Granger Causality** | Test if past crude oil returns predict current stock returns | Crude oil does NOT Granger-cause stock returns |
| **Engle-Granger Cointegration** | Test for long-run equilibrium between price levels | No cointegration |
| **OLS Regression** | Quantify crude oil sensitivity (beta) controlling for market and FX | Beta coefficient = 0 |
| **Lagged Regression** | Test predictive power of past oil returns on current stock returns | Lagged beta = 0 |
| **VAR Model** | Capture dynamic feedback between crude oil and stock returns | No dynamic interaction |
| **Impulse Response Functions** | Trace the effect of a shock in crude oil on stock returns over time | N/A (derived from VAR) |
| **Event Study (CAR)** | Measure abnormal stock returns around major oil price shocks | CAR = 0 (no abnormal return) |

### 3.3 Software and Libraries

- **Python 3.12.4** with the following key packages:
  - `pandas`, `numpy` - Data manipulation
  - `matplotlib`, `seaborn` - Visualization
  - `statsmodels` - ADF test, Granger causality, cointegration, OLS, VAR
  - `scipy` - Statistical testing
  - `kiteconnect` - Zerodha Kite API client
  - `fredapi` - FRED data access
  - `yfinance` - Yahoo Finance data

---

## 4. Exploratory Data Analysis

### 4.1 Crude Oil Price Trends (Figure 1)

**Visualization**: `fig_1_crude_oil_prices.png`

The chart displays crude oil prices from multiple sources with a dual y-axis:
- **Left axis (INR)**: MCX Crude Oil futures in Indian Rupees per barrel
- **Right axis (USD)**: Brent and WTI crude futures in US Dollars per barrel

**Key observations**:
- Brent crude ranged from approximately $63 to $82 per barrel during the study period.
- WTI crude tracked below Brent by $3-5/barrel, consistent with the historical Brent-WTI spread.
- MCX Crude Oil prices reflect the INR-equivalent, which is influenced by both international crude and USD/INR movements.
- USD/INR exchange rate trended from ~85.8 at the start to ~90.6 by end of the period, implying a ~5.6% INR depreciation. This amplified the crude oil cost impact for Indian companies.

**Interpretation**: The INR depreciation during the study period meant that even when international crude prices moderated, the rupee-denominated cost for Indian companies did not decline proportionally. This dual effect (commodity price + currency) is a critical factor for Indian transportation firms.

### 4.2 Transportation Stock Performance (Figure 2)

**Visualization**: `fig_2_transportation_stocks.png`

All stock prices are normalized to a base of 100 at the start of the study period to enable direct performance comparison.

**Key observations by sector**:

- **Airlines (INDIGO)**: Showed moderate price appreciation (~9% by period end), with intermittent pullbacks. Performance was volatile, reflecting sensitivity to fuel costs and seasonal demand patterns.
- **Logistics**: Mixed performance. VRL Logistics (+18%) and Mahindra Logistics (+14%) outperformed, while TCI Express (-31%) and Blue Dart (-15%) underperformed significantly.
- **Shipping**: SCI (+28%) and Great Eastern Shipping (+37%) were among the strongest performers, benefiting from rising freight rates and shipping demand.
- **Oil Marketing Companies**: IOC (+36%), BPCL (+37%), and HINDPETRO (+15%) all showed positive returns, driven by marketing margin improvements and inventory gains.

### 4.3 Crude Oil vs. Sector Comparison (Figure 3)

**Visualization**: `fig_3_crude_vs_sectors.png`

Three-panel chart showing normalized crude oil prices overlaid with:
1. **Airlines**: INDIGO moved inversely to crude oil at times, consistent with the cost-pressure narrative.
2. **Logistics**: Mixed patterns; VRL Logistics showed positive co-movement while MAHLOG showed divergence.
3. **Oil Marketing Companies**: IOC, HINDPETRO, and BPCL showed patterns that were not strongly synchronous with crude, reflecting the complex margin dynamics of regulated pricing.

---

## 5. Correlation Analysis

### 5.1 Pearson and Spearman Correlations (Figure 4)

**Visualization**: `fig_4_correlations.png`

#### Results Table: Crude Oil vs. Transportation Stock Returns

| Stock | Pearson Correlation | Spearman Correlation | Direction |
|-------|:-------------------:|:--------------------:|:---------:|
| INDIGO | **-0.1501** | -0.1378 | Negative |
| MAHLOG | **-0.1314** | -0.1245 | Negative |
| IOC | **-0.0992** | +0.0157 | Negative (Pearson) / Weak Positive (Spearman) |
| HINDPETRO | **-0.0851** | +0.0939 | Negative (Pearson) / Weak Positive (Spearman) |
| BPCL | **-0.0309** | +0.1071 | Negative (Pearson) / Positive (Spearman) |
| GESHIP | **-0.0207** | +0.0091 | Near zero |
| BLUEDART | **+0.0536** | +0.0351 | Weak Positive |
| SCI | **+0.0758** | +0.0553 | Weak Positive |
| TCIEXP | **+0.1183** | +0.0012 | Positive (Pearson) / Near zero (Spearman) |
| VRLLOG | **+0.1339** | +0.1800 | Positive |

**Summary Statistics**:
- **Average Pearson Correlation**: -0.0137 (near zero, slightly negative)
- **Positive Correlations**: 4 out of 10 stocks
- **Negative Correlations**: 6 out of 10 stocks
- **Strongest Negative**: INDIGO (-0.15) - the airline stock, as theoretically expected
- **Strongest Positive**: VRLLOG (+0.13) - VRL Logistics

**Interpretation**:
- The correlations are generally **weak** (all below |0.15|), indicating that crude oil returns explain only a small fraction of daily stock return variation on their own.
- **INDIGO** shows the strongest negative correlation, aligning with the expectation that airlines are hurt by rising fuel costs.
- The **divergence between Pearson and Spearman** for OMCs (IOC, HINDPETRO, BPCL) is notable: the linear relationship is slightly negative while the rank-based relationship is slightly positive. This suggests the relationship may be non-linear or driven by outliers.
- **VRLLOG** showing a positive correlation is somewhat counterintuitive for a logistics firm and may reflect company-specific factors or indirect effects.

### 5.2 Rolling Correlation (Figure 5)

**Visualization**: `fig_5_rolling_correlation.png`

A 90-day rolling window correlation reveals that the commodity-equity relationship is **time-varying and unstable**:

- Correlations fluctuate between approximately -0.4 and +0.4 over the study period.
- There are periods of strong negative correlation (as expected) alternating with periods of positive correlation.
- This instability suggests that simple static correlation measures may understate the complexity of the relationship.

### 5.3 Full Correlation Heatmap (Figure 6)

**Visualization**: `fig_6_correlation_heatmap.png`

The full correlation heatmap covers returns across all variables (crude oil variants, stocks, indices, FX, macro). Key observations:

- **Intra-sector correlations** are higher than commodity-equity correlations. For example, OMC stocks are highly correlated with each other.
- **Market beta dominates**: Most stocks show stronger correlation with NIFTY50/NIFTY AUTO than with crude oil, indicating that broad market movements are a more powerful driver than commodity prices.
- **Brent and WTI** are near-perfectly correlated (~0.95+), confirming they can be used interchangeably.

### 5.4 Sector-Wise Correlation Summary (Figure 7)

**Visualization**: `fig_7_sector_correlation.png`

| Sector | Avg Correlation with Crude | Avg Mean Return | Avg Std Dev | Avg Sharpe Ratio |
|--------|:--------------------------:|:---------------:|:-----------:|:----------------:|
| **Airlines (INDIGO)** | -0.1501 | 0.0003 | 0.0170 | 0.286 |
| **Logistics** | -0.0060 | -0.0002 | 0.0200 | -0.158 |
| **Shipping** | +0.0276 | 0.0010 | 0.0241 | 0.681 |
| **Oil Marketing** | -0.0717 | 0.0008 | 0.0169 | 0.854 |

**Interpretation**:
- **Airlines** have the most negative correlation with crude oil and the lowest Sharpe ratio among sectors, reflecting fuel cost vulnerability.
- **Shipping** companies have a slight positive correlation, potentially because rising oil prices coincide with increased global trade activity and higher freight rates.
- **Oil Marketing** companies show a negative Pearson correlation on average, somewhat counterintuitively. This may be because regulated fuel pricing in India means OMCs cannot always pass through cost increases.
- **Logistics** has near-zero average correlation, suggesting mixed and offsetting effects within the sector.

---

## 6. Statistical Testing

### 6.1 Stationarity Tests - Augmented Dickey-Fuller (ADF)

**Purpose**: Verify that return series are stationary, a prerequisite for valid correlation analysis, Granger causality tests, and VAR modeling.

**Hypothesis**:
- H0: The series has a unit root (non-stationary)
- H1: The series is stationary

#### Results

| Variable | ADF Statistic | p-value | Critical Value (5%) | Stationary |
|----------|:------------:|:-------:|:-------------------:|:----------:|
| Crude Oil | -12.804 | 6.65e-24 | -2.884 | **Yes** |
| INDIGO | -16.882 | 1.06e-29 | -2.872 | **Yes** |
| BLUEDART | -15.157 | 6.67e-28 | -2.872 | **Yes** |
| VRLLOG | -6.427 | 1.73e-08 | -2.872 | **Yes** |
| TCIEXP | -12.732 | 9.34e-24 | -2.872 | **Yes** |
| MAHLOG | -4.429 | 2.64e-04 | -2.872 | **Yes** |
| SCI | -4.145 | 8.15e-04 | -2.872 | **Yes** |
| GESHIP | -5.972 | 1.92e-07 | -2.872 | **Yes** |
| IOC | -16.994 | 8.82e-30 | -2.872 | **Yes** |
| HINDPETRO | -16.824 | 1.17e-29 | -2.872 | **Yes** |
| BPCL | -17.612 | 3.87e-30 | -2.872 | **Yes** |

**Result**: All 11 return series (1 crude oil + 10 stocks) are **stationary** at the 5% significance level (all p-values < 0.001). The ADF statistics are far below the critical values, providing strong evidence against unit roots. This confirms that the log return transformations successfully achieved stationarity, validating the subsequent time-series analyses.

### 6.2 Granger Causality Tests

**Purpose**: Test whether past values of crude oil returns contain information useful for predicting current stock returns, beyond what is already contained in the stock's own past returns.

**Hypothesis**:
- H0: Crude oil does NOT Granger-cause stock returns
- H1: Crude oil Granger-causes stock returns

**Configuration**: Maximum lag tested = 10 days. Optimal lag selected by minimum p-value across F-tests.

#### Forward Causality: Crude Oil → Stocks

| Crude Oil → Stock | Optimal Lag | p-value | Granger Causes? |
|-------------------|:----------:|:-------:|:---------------:|
| Crude → **VRLLOG** | 2 | **0.0318** | **Yes** |
| Crude → **MAHLOG** | 1 | **0.0425** | **Yes** |
| Crude → **SCI** | 1 | **0.0498** | **Yes** |
| Crude → INDIGO | 1 | 0.0895 | No |
| Crude → BPCL | 9 | 0.1533 | No |
| Crude → BLUEDART | 5 | 0.1615 | No |
| Crude → HINDPETRO | 1 | 0.2457 | No |
| Crude → IOC | 9 | 0.3601 | No |
| Crude → TCIEXP | 10 | 0.3959 | No |
| Crude → GESHIP | 1 | 0.5635 | No |

**Key Finding**: Crude oil Granger-causes **3 out of 10 stocks** at the 5% significance level:
1. **VRLLOG** (VRL Logistics) - 2-day lag, p = 0.032
2. **MAHLOG** (Mahindra Logistics) - 1-day lag, p = 0.043
3. **SCI** (Shipping Corp of India) - 1-day lag, p = 0.050

INDIGO narrowly misses significance at 5% (p = 0.090) but is significant at the 10% level.

**Interpretation**: The evidence for predictive causality is strongest for **logistics and shipping** stocks. The 1-2 day lag is economically sensible: it takes approximately one trading day for crude oil information to be reflected in these stocks, likely due to the time required for market participants to process the commodity price signal and its implications for transportation costs.

#### Reverse Causality: Stocks → Crude Oil

Reverse causality tests were conducted to verify the directionality. No significant reverse causality (stocks causing crude oil) was found, confirming that the causal direction flows **from crude oil to stocks**, not the other way around. This is economically logical: Indian transportation stocks are too small to influence global crude oil prices.

### 6.3 Cointegration Tests (Engle-Granger)

**Purpose**: Test whether crude oil prices and stock prices share a long-run equilibrium relationship (i.e., they move together in levels over the long term, even if they diverge temporarily).

**Hypothesis**:
- H0: No cointegration (no long-run equilibrium)
- H1: Series are cointegrated

#### Results

| Stock | Test Statistic | p-value | Cointegrated? |
|-------|:--------------:|:-------:|:-------------:|
| VRLLOG | -2.329 | 0.359 | No |
| INDIGO | -2.267 | 0.390 | No |
| HINDPETRO | -2.193 | 0.428 | No |
| TCIEXP | -2.099 | 0.477 | No |
| BLUEDART | -2.043 | 0.506 | No |
| BPCL | -2.031 | 0.512 | No |
| IOC | -2.006 | 0.526 | No |
| SCI | -1.983 | 0.537 | No |
| MAHLOG | -1.859 | 0.601 | No |
| GESHIP | -1.573 | 0.732 | No |

**Result**: **No cointegration** was found between crude oil and any of the 10 transportation stocks at the 5% significance level. All p-values exceed 0.35.

**Interpretation**: The absence of cointegration implies that there is **no long-run equilibrium** binding crude oil prices and these stock prices together. While there may be short-run correlations and lead-lag effects (as shown by Granger causality), the price levels do not converge to a stable long-term relationship. This is consistent with the view that stock prices are driven by many factors beyond commodity prices (earnings growth, market sentiment, sector rotation, company-specific news), and the crude oil effect is one of many inputs rather than a dominant long-run anchor.

**Caveat**: The 13-month study period may be too short to detect cointegration, which typically requires multi-year data to manifest.

---

## 7. Regression Analysis

### 7.1 OLS Regression Model

**Model Specification**:

```
Stock Return_i,t = α + β₁(Crude Oil Return_t) + β₂(NIFTY50 Return_t) + β₃(USD/INR Return_t) + ε_i,t
```

Where:
- `α` = intercept (alpha, abnormal return)
- `β₁` = crude oil sensitivity (the parameter of primary interest)
- `β₂` = market beta (controls for broad market movements)
- `β₃` = currency sensitivity (controls for INR depreciation/appreciation effects)
- `ε` = error term

#### Results

| Stock | Alpha | Beta Crude | p-value (Crude) | Sig | Beta Market | p-value (Market) | Beta FX | p-value (FX) | R² | Adj R² |
|-------|:-----:|:----------:|:---------------:|:---:|:-----------:|:----------------:|:-------:|:------------:|:--:|:------:|
| INDIGO | -0.0019 | **-0.1114** | 0.156 | | 1.4277 | 9.42e-09 | -0.1041 | 0.809 | 0.272 | 0.254 |
| BLUEDART | -0.0002 | +0.0886 | 0.389 | | 0.7408 | 0.016 | -0.7173 | 0.206 | 0.080 | 0.058 |
| VRLLOG | +0.0002 | **+0.1443** | 0.053 | * | 0.8798 | 1.06e-04 | +0.0986 | 0.809 | 0.136 | 0.115 |
| TCIEXP | -0.0019 | **+0.1731** | 0.082 | * | 1.1867 | 9.39e-05 | +0.3996 | 0.465 | 0.129 | 0.108 |
| MAHLOG | +0.0011 | -0.1505 | 0.208 | | 1.1457 | 0.002 | -0.2710 | 0.680 | 0.108 | 0.087 |
| SCI | +0.0010 | +0.1782 | 0.211 | | 1.8199 | 3.14e-05 | +0.9632 | 0.220 | 0.134 | 0.113 |
| GESHIP | +0.0019 | +0.0159 | 0.842 | | 1.3139 | 1.58e-07 | +0.4369 | 0.321 | 0.200 | 0.181 |
| IOC | +0.0014 | -0.0482 | 0.435 | | 1.0809 | 3.06e-08 | +0.5199 | 0.128 | 0.225 | 0.207 |
| HINDPETRO | +0.0004 | -0.0547 | 0.512 | | 1.2056 | 3.25e-06 | +0.7748 | 0.094 | 0.165 | 0.145 |
| BPCL | +0.0010 | +0.0055 | 0.936 | | 1.2483 | 1.10e-08 | +0.9239 | 0.016 | 0.231 | 0.213 |

*Significance levels: \*\*\* p<0.01, \*\* p<0.05, \* p<0.10*

#### Key Findings from Regression Analysis

**1. Crude Oil Beta (β₁)**:
- Average crude oil beta across all stocks: **+0.024** (near zero)
- Range: -0.151 (MAHLOG) to +0.178 (SCI)
- Only 2 stocks (VRLLOG, TCIEXP) show marginally significant crude oil betas at the 10% level
- **Crude oil is NOT a statistically significant predictor** of most transportation stock returns after controlling for market and currency effects

**2. Market Beta (β₂)**:
- All stocks show **highly significant positive market betas** (all p-values < 0.02)
- Market betas range from 0.74 (BLUEDART) to 1.82 (SCI)
- SCI has the highest market beta (1.82), indicating it is the most sensitive to broad market movements
- **The market factor dominates**: NIFTY50 returns explain far more variation in stock returns than crude oil returns

**3. Currency Beta (β₃)**:
- USD/INR sensitivity is generally positive but mostly insignificant
- Only BPCL shows a statistically significant FX beta (+0.924, p=0.016), suggesting that a 1% INR depreciation is associated with a ~0.92% increase in BPCL returns
- This positive FX sensitivity for OMCs is consistent with inventory revaluation gains when the rupee weakens

**4. Model Fit (R²)**:
- Average R²: **0.168** (range: 0.080 to 0.272)
- Best-fit models: INDIGO (27.2%), BPCL (23.1%), IOC (22.5%)
- Worst-fit: BLUEDART (8.0%)
- The low R² values indicate that the three-factor model explains only a modest portion of daily return variation, which is typical for daily equity return models

### 7.2 Lagged Regression Analysis (Figure 8)

**Visualization**: `fig_8_regression_coefficients.png`

Lagged regressions test whether past crude oil returns (lagged by 1-10 days) predict current stock returns:

```
Stock Return_t = α + β(Crude Oil Return_{t-k}) + ε_t,  for k = 0, 1, 2, ..., 10
```

**Key finding**: For most stocks, the contemporaneous (lag=0) or lag-1 relationship shows the highest explanatory power, with R² declining rapidly for higher lags. This implies that crude oil information is priced into stocks within **1-2 trading days**, consistent with semi-strong market efficiency.

---

## 8. Event Study Analysis

### 8.1 Identification of Oil Price Shocks (Figure 9)

**Visualization**: `fig_9_oil_shocks.png`

**Methodology**: Oil price shocks were identified as single-day crude oil returns exceeding +/-5% (|r| > 0.05). This threshold captures extreme moves that are likely to generate significant market attention and prompt reassessment of fuel-cost-sensitive stocks.

The chart overlays these shock events on the crude oil price time series, showing their timing and magnitude.

### 8.2 Cumulative Abnormal Returns (CAR) Analysis (Figure 10)

**Visualization**: `fig_10_event_study_car.png`

**Methodology**:
1. The top 3 largest shocks (by absolute return) were selected as event dates.
2. For each event, a window of [-5, +5] trading days was defined.
3. Abnormal returns were calculated as: `AR_t = Stock Return_t - Market Return_t` (market-adjusted model)
4. Cumulative abnormal returns (CAR) were summed over the event window.

**Interpretation**: The CAR analysis reveals how stocks react in the days surrounding major crude oil price shocks:
- **Negative CAR** for a stock means it underperformed the market around oil shocks, suggesting vulnerability to crude oil volatility.
- **Positive CAR** means the stock outperformed the market, suggesting resilience or even a benefit from oil price dislocations.

The bar chart displays average CAR across the top 3 shock events for each stock, providing a visual ranking of oil shock vulnerability.

---

## 9. VAR Model & Impulse Response

### 9.1 Vector Autoregression (VAR) Model

**Purpose**: Capture the dynamic bidirectional feedback between crude oil returns and the most correlated stock's returns using a system-of-equations approach.

**Model**:
```
Crude_t     = Σ α₁ᵢ Crude_{t-i} + Σ β₁ᵢ Stock_{t-i} + ε₁_t
Stock_t     = Σ α₂ᵢ Crude_{t-i} + Σ β₂ᵢ Stock_{t-i} + ε₂_t
```

**Lag Selection**: Optimal lag order was determined using the Akaike Information Criterion (AIC) via `var_model.select_order(maxlags=10)`.

### 9.2 Impulse Response Functions (Figure 11)

**Visualization**: `fig_11_var_irf.png` (Note: Figure 11 was not generated if VAR model encountered issues)

The IRF analysis traces how a one-standard-deviation shock to crude oil returns propagates to stock returns over the subsequent 10 trading days.

**Four panels**:
1. **Crude Oil → Crude Oil**: Shows persistence of oil price shocks (how long an oil shock continues to affect future oil returns).
2. **Crude Oil → Stock**: The primary panel of interest. Shows how a crude oil shock affects the selected stock over time.
3. **Stock → Crude Oil**: Tests reverse transmission (expected to be near zero).
4. **Stock → Stock**: Shows persistence of stock return shocks.

**Expected pattern**: A crude oil shock should cause a transient response in the stock (peaking at lag 1-2 and decaying to zero by lag 5-10), with no significant reverse effect.

---

## 10. Volatility Regime Analysis

### 10.1 Methodology (Figure 12)

**Visualization**: `fig_12_volatility_analysis.png`

Crude oil return volatility was calculated using a 30-day rolling window, annualized by multiplying by sqrt(252). The median volatility was used as the threshold to split the sample into **high volatility** and **low volatility** regimes. Correlations between crude oil and each stock were then computed separately for each regime.

### 10.2 Results

| Stock | High Vol Correlation | Low Vol Correlation | Difference |
|-------|:--------------------:|:-------------------:|:----------:|
| TCIEXP | **+0.244** | -0.082 | **+0.326** |
| IOC | -0.054 | -0.172 | +0.118 |
| SCI | +0.105 | +0.005 | +0.100 |
| VRLLOG | +0.169 | +0.075 | +0.095 |
| BPCL | -0.023 | -0.049 | +0.026 |
| HINDPETRO | -0.128 | -0.039 | -0.089 |
| MAHLOG | -0.170 | -0.067 | -0.104 |
| INDIGO | **-0.255** | -0.056 | **-0.199** |
| GESHIP | -0.120 | +0.072 | -0.192 |
| BLUEDART | -0.021 | +0.221 | **-0.242** |

#### Key Findings

1. **INDIGO's negative correlation intensifies in high-volatility periods** (-0.255 vs. -0.056). This means that when crude oil is most volatile, IndiGo's stock is most adversely affected. This is economically intuitive: volatile oil prices create uncertainty about airline margins, and the market penalizes the stock more.

2. **TCIEXP shows the largest regime shift** (+0.244 in high vol vs. -0.082 in low vol, a difference of +0.326). TCI Express becomes more positively correlated with crude during turbulent periods, which may reflect speculative trading or sectoral repositioning.

3. **BLUEDART flips sign** (-0.021 in high vol vs. +0.221 in low vol), suggesting that the crude oil-Blue Dart relationship is fundamentally different depending on the market environment.

4. **Average pattern**: The correlation tends to become more negative (or less positive) during high-volatility periods for airlines and logistics, consistent with the "cost pressure" hypothesis being amplified during uncertain times.

---

## 11. Sub-Period Analysis

### 11.1 Period Definitions (Figure 13)

**Visualization**: `fig_13_period_analysis.png`

The study data covers only the **Full Period** (January 2025 - February 2026). The sub-period analysis was designed to test structural breaks across years (2022, 2023, 2024), but since the actual data collection yielded data only for the 2025-2026 window, the sub-period comparison was limited to the full sample.

#### Full Period Correlations (130 observations)

| Stock | Correlation with Crude Oil |
|-------|:--------------------------:|
| INDIGO | -0.1501 |
| MAHLOG | -0.1314 |
| IOC | -0.0992 |
| HINDPETRO | -0.0851 |
| BPCL | -0.0309 |
| GESHIP | -0.0207 |
| BLUEDART | +0.0536 |
| SCI | +0.0758 |
| TCIEXP | +0.1183 |
| VRLLOG | +0.1339 |

**Note**: The planned multi-year comparison (2022 Russia-Ukraine crisis, 2023, 2024) would require an extended data collection period. This remains a recommendation for future work.

---

## 12. Key Findings & Conclusions

### 12.1 Summary of Statistical Results

| Analysis | Key Result |
|----------|-----------|
| **Correlation** | Weak correlations overall (avg Pearson: -0.014). INDIGO most negatively correlated (-0.15). VRLLOG most positively (+0.13). |
| **ADF Tests** | All 11 return series are stationary (all p < 0.001). Analysis prerequisites met. |
| **Granger Causality** | Crude oil Granger-causes 3/10 stocks (VRLLOG, MAHLOG, SCI at 5% level). No reverse causality found. |
| **Cointegration** | No cointegration detected for any stock pair (all p > 0.35). No long-run equilibrium. |
| **OLS Regression** | Crude oil beta mostly insignificant after controlling for market (NIFTY50). Average R² = 0.168. Market beta dominates. |
| **Volatility Regimes** | Correlations shift across volatility regimes. INDIGO correlation intensifies from -0.06 to -0.26 in high-vol periods. |

### 12.2 Core Conclusions

**1. The crude oil-equity linkage exists but is weak in isolation.**
Daily return correlations between MCX crude oil and Indian transportation stocks are statistically weak (all |r| < 0.15). Crude oil explains at most ~2% of daily stock return variance on its own (r² = 0.15² = 0.02).

**2. Market risk dominates commodity risk.**
In the multivariate OLS regression, the NIFTY50 market beta is highly significant for all 10 stocks (all p < 0.02), while the crude oil beta is significant for only 2 stocks at 10%. This indicates that broad market movements overwhelm the commodity-specific signal on a daily basis.

**3. Crude oil has predictive power for select stocks.**
Granger causality tests show that past crude oil returns contain statistically significant predictive information for 3 stocks: VRL Logistics (2-day lag), Mahindra Logistics (1-day lag), and Shipping Corporation of India (1-day lag). This suggests partial informational inefficiency that could theoretically be exploited.

**4. No long-run equilibrium relationship.**
The absence of cointegration across all 10 stock pairs implies that crude oil and these stocks do not share a stable long-run relationship. Their prices may drift apart permanently, and crude oil cannot be used as a long-term anchor for transportation stock valuation.

**5. The relationship is regime-dependent.**
Volatility regime analysis reveals that correlations are not constant. Airlines (INDIGO) show significantly stronger negative correlation during high-volatility periods, while several logistics stocks flip sign across regimes. This has practical implications for hedging: the effectiveness of commodity-based hedges varies with market conditions.

**6. Sector heterogeneity is significant.**
Airlines, logistics, shipping, and OMCs each exhibit distinct correlation patterns. A one-size-fits-all approach to the commodity-equity linkage would be misleading; sector-specific analysis is essential.

### 12.3 Practical Implications

- **Portfolio Managers**: The weak correlations suggest limited diversification benefit from pairing crude oil and Indian transportation stocks. However, the regime-dependent nature means that during periods of crude oil turbulence, the hedging relationship becomes more pronounced and potentially useful.
- **Risk Managers**: Airlines and logistics firms should monitor crude oil volatility as a leading indicator of increased equity risk, especially during high-volatility regimes.
- **Traders**: The Granger causality results (1-2 day lag) suggest a short-lived predictive signal. Crude oil price movements may provide a 1-2 day lead for VRLLOG, MAHLOG, and SCI.
- **Corporate Strategy**: Indian transportation companies face a currency-amplified crude oil risk. The 5.6% INR depreciation during the study period magnified fuel cost impacts.

---

## 13. Limitations & Future Work

### 13.1 Limitations

1. **Short time period**: The 13-month window (Jan 2025 - Feb 2026) limits the ability to detect long-run relationships and structural breaks. Cointegration tests, in particular, are more powerful over multi-year horizons.
2. **Daily frequency**: Intraday dynamics are missed. Crude oil prices may affect stock prices within the same trading day, which is captured by contemporaneous correlation but not by Granger causality.
3. **Limited stock universe**: Only 10 stocks were analyzed. Expanding to broader transportation, aviation, and energy indices would provide more robust sectoral conclusions.
4. **Linear models**: The analysis relies on linear methods (Pearson correlation, OLS, VAR). Non-linear relationships, threshold effects, or asymmetric responses (different effects for oil price increases vs. decreases) are not captured.
5. **No control for company-specific events**: Earnings announcements, management changes, regulatory actions, and other idiosyncratic factors are not controlled for in the regression models.
6. **MCX data gaps**: MCX crude oil futures data had gaps due to contract rollovers. Brent crude was used as the primary benchmark where MCX data was unavailable.

### 13.2 Recommendations for Future Research

1. **Extend the time period** to 5-10 years to capture multiple crude oil cycles (including COVID-19 crash, Russia-Ukraine price spike, OPEC+ cuts).
2. **Asymmetric models**: Implement NARDL (Non-linear ARDL) or threshold regression to test if the relationship differs for oil price increases vs. decreases.
3. **GARCH models**: Use DCC-GARCH (Dynamic Conditional Correlation GARCH) to formally model time-varying correlations and volatility spillovers.
4. **Intraday analysis**: Use tick-level or hourly data to capture same-day transmission mechanisms.
5. **Panel regression**: Pool all 10 stocks in a panel framework with fixed effects to gain statistical power.
6. **Fundamental analysis**: Incorporate fuel cost as a percentage of revenue, hedging policies, and pricing power as company-level moderating variables.
7. **Global comparison**: Compare the Indian commodity-equity linkage with other emerging markets (Brazil, Turkey, South Africa) to test generalizability.

---

## 14. Appendix: Output Files

### 14.1 Data Files

| File | Description |
|------|-------------|
| `data_prices.csv` | Master price dataset (291 rows x 20 columns) |
| `data_returns.csv` | Log returns dataset |

### 14.2 Results Files

| File | Description |
|------|-------------|
| `results_correlation.csv` | Pearson and Spearman correlations |
| `results_sector_stats.csv` | Sector-wise descriptive statistics |
| `results_adf_test.csv` | ADF stationarity test results |
| `results_granger_causality.csv` | Granger causality test results |
| `results_cointegration.csv` | Engle-Granger cointegration test results |
| `results_regression.csv` | OLS regression coefficients and fit statistics |
| `results_volatility_regime.csv` | High vs. low volatility regime correlations |
| `results_period_analysis.csv` | Sub-period correlation analysis |

### 14.3 Figures

| Figure | File | Description |
|--------|------|-------------|
| Fig 1 | `fig_1_crude_oil_prices.png` | Crude oil prices (MCX, Brent, WTI) and USD/INR |
| Fig 2 | `fig_2_transportation_stocks.png` | Normalized stock performance (base=100) |
| Fig 3 | `fig_3_crude_vs_sectors.png` | Crude oil vs. airlines, logistics, OMCs |
| Fig 4 | `fig_4_correlations.png` | Pearson and Spearman correlation bar charts |
| Fig 5 | `fig_5_rolling_correlation.png` | 90-day rolling correlation time series |
| Fig 6 | `fig_6_correlation_heatmap.png` | Full correlation heatmap |
| Fig 7 | `fig_7_sector_correlation.png` | Sector-wise average correlation |
| Fig 8 | `fig_8_regression_coefficients.png` | OLS crude oil beta and R² |
| Fig 9 | `fig_9_oil_shocks.png` | Oil price shocks on price chart |
| Fig 10 | `fig_10_event_study_car.png` | Cumulative abnormal returns around shocks |
| Fig 12 | `fig_12_volatility_analysis.png` | Volatility regime analysis |
| Fig 13 | `fig_13_period_analysis.png` | Sub-period correlation comparison |

---

*Documentation generated from analysis in `Commodity_Analysis.ipynb`*
*Study Period: January 2025 - February 2026*
*Author: Chinmay Dongarkar*
