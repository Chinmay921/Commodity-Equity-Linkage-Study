# Commodity-Equity Linkage Study

**Analyzing the Relationship between Crude Oil Prices and Indian Transportation Stocks**

![Python](https://img.shields.io/badge/Python-3.12-blue)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-green)
![Status](https://img.shields.io/badge/Status-Active-success)

---

## Overview

This research investigates how crude oil price fluctuations impact Indian transportation and energy sector equities. Using advanced econometric techniques, the study examines whether oil price movements systematically affect stock returns of companies with high fuel cost exposure.

**Key Questions:**
- Do crude oil prices Granger-cause stock returns?
- Is the relationship symmetric across sectors (airlines vs. logistics vs. OMCs)?
- Does correlation strength vary with market volatility?
- Are there long-run equilibrium relationships (cointegration)?

---

## Project Structure

```
├── Commodity_Analysis.ipynb          # Main analysis notebook
├── Research_Documentation.md         # Detailed methodology & findings
├── data_prices.csv                   # Historical price data
├── data_returns.csv                  # Daily returns
├── results_*.csv                     # Statistical test results
└── fig_*.png                         # Visualizations (13 figures)
```

---

## 🔬 Methodology

**Data Sources:**
- MCX Crude Oil Futures
- Brent & WTI Crude (Yahoo Finance)
- NSE Stock Data via Kite API
- FRED Economic Indicators (VIX, DXY)

**Analytical Techniques:**
1. **Correlation Analysis** - Pearson, Spearman, rolling windows
2. **Granger Causality Tests** - Lead-lag relationships
3. **Cointegration Testing** - Long-run equilibrium (Engle-Granger, Johansen)
4. **Regression Analysis** - OLS with market & FX controls
5. **Event Study** - Cumulative abnormal returns around oil shocks
6. **VAR/Impulse Response** - Dynamic spillover effects
7. **Volatility Regime Analysis** - High vs. low volatility periods

---

## Key Findings

### Sector-Specific Results:

| Sector | Correlation | Granger Causality | Key Insight |
|--------|------------|-------------------|-------------|
| **Airlines** | Negative (-0.15 to -0.30) | Yes (oil → stock) | Strong inverse relationship |
| **Logistics** | Negative (-0.20 to -0.35) | Yes | Fuel costs dominate |
| **OMCs** | Positive (+0.40 to +0.60) | Yes | Inventory gains benefit |

**Notable Discoveries:**
- Correlation strengthens during high volatility periods
- Asymmetric response: Oil shocks impact stocks more than gradual changes
- Long-run cointegration detected for OMC stocks

---

## Technologies Used

- **Python 3.12**: Core language
- **Pandas/NumPy**: Data manipulation
- **Statsmodels**: Econometric testing (ADF, Granger, VAR)
- **Matplotlib/Seaborn**: Visualization
- **yFinance**: Market data
- **KiteConnect API**: Indian stock data


---

## Citation

If you use this research, please cite:
```
Dongarkar, C. (2026). Commodity-Equity Linkage Study: Analyzing the Relationship 
between Crude Oil Prices and Indian Transportation Stocks. GitHub Repository.
```

---

## Contact

**Author:** Chinmay Dongarkar  
**Institution:** SSODL & DTU  

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🔗 References

See [Research_Documentation.md](Research_Documentation.md) for complete methodology, statistical results, and academic references.
