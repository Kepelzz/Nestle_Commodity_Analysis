# Nestle_Commodity_Analysis
End-to-end commodity price analysis: World Bank + Alpha Vantage pipelines, correlation analysis across 427 months, OLS regression identifying cocoa as Nestlé's most stock-sensitive commodity. Python · pandas · statsmodels · plotly

# Nestlé Commodity vs Stock Performance

End-to-end data pipeline and statistical analysis examining whether Nestlé's key commodity input prices have a measurable relationship with Nestlé's stock price (NESN.SW on the Swiss Exchange).

**427 months of data · Jan 1990 – Feb 2026 · 8 commodities · 2 data sources · OLS regression**

---

## Overview

Nestlé's cost base depends heavily on raw commodities — coffee, cocoa, sugar, wheat, corn, palm oil, soybeans. The question this project explores: **do movements in these commodity prices actually move Nestlé's stock?**

The pipeline covers three phases:

1. **Data Engineering** — two separate pipelines for sourcing commodity prices (World Bank Excel + Alpha Vantage API), cleaning, unit conversion, and building a star schema ready for Power BI
2. **Exploratory Analysis** — correlation heatmap, lagged correlation (0–6 months), inter-commodity clustering
3. **Statistical Modelling** — OLS regression identifying which commodities explain stock return variation

---

## Key Findings

- **Cocoa is the only commodity with a meaningful negative correlation** with Nestlé's stock returns (r = −0.16). The market reaction is immediate — same month, no lag
- **Coffee shows a slight positive signal at lag_1m** — rising coffee prices likely signal strong global demand, which benefits Nespresso and Nescafé revenue
- **R² = 3.8%** — two commodities explain ~4% of monthly stock return variation. The remaining 96% is driven by macro conditions, FX (CHF/EUR), earnings surprises, and broader market sentiment
- **Nestlé's diversification absorbs most commodity shocks** — 400+ brands across categories prevent any single input from dominating financial performance

---

## Notebooks

### `01_World_Bank_Pipeline.ipynb`
Full historical pipeline using the World Bank Pink Sheet — the gold standard for commodity data, covering 71 commodities from 1960.

- Cleans non-standard Excel layout (metadata rows, custom date format)
- Filters to 8 Nestlé-relevant commodities
- Reshapes wide → long format via `pd.melt()`
- Engineers features: YoY % change, 12-month rolling average
- Builds star schema (`dim_commodity`, `fact_prices`) exported as CSV for Power BI
- Downloads Nestlé stock price via `yfinance`, merges with commodity data
- Runs correlation analysis, lagged correlation, and OLS regression

### `02_Alpha_Vantage_Pipeline.ipynb`
Companion pipeline using the Alpha Vantage REST API — designed to top up recent months when the Pink Sheet hasn't been updated yet.

- Reusable `fetch_commodity()` function with built-in error handling and rate limit management
- Converts raw API units (cents/lb) to $/kg and $/mt to match World Bank schema
- Handles Alpha Vantage quirks: `.` as null placeholder, newest-first sort order
- Visualises indexed price trends — all commodities normalised to 100 for fair comparison

**Recommended architecture:**
```
World Bank (1960 → last month)  +  Alpha Vantage (last 1–2 months top-up)
                        ↓
              merge on (date, commodity)
```

---

## Commodities Covered

| Commodity | Category | Unit | Pipeline |
|-----------|----------|------|----------|
| Coffee Arabica | Beverages | $/kg | Both |
| Coffee Robusta | Beverages | $/kg | World Bank |
| Cocoa | Confectionery | $/kg | World Bank |
| Sugar (World) | Sweeteners | $/kg | Both |
| Wheat (US SRW) | Grains | $/mt | Both |
| Corn (Maize) | Grains | $/mt | Both |
| Palm Oil | Oils & Fats | $/mt | World Bank |
| Soybeans | Oils & Fats | $/mt | World Bank |

---

## Tech Stack

```
Python 3.9
├── pandas          # data wrangling, reshaping, feature engineering
├── yfinance        # Nestlé stock price download
├── requests        # Alpha Vantage REST API
├── scikit-learn    # linear regression
├── statsmodels     # OLS with significance testing
├── seaborn / matplotlib  # static visualisation
└── plotly          # interactive indexed price chart
```

---

## Project Structure

```
nestle-commodity-analysis/
│
├── notebooks/
│   ├── 01_World_Bank_Pipeline.ipynb
│   └── 02_Alpha_Vantage_Pipeline.ipynb
│
├── data/
│   ├── dim_commodity.csv       # dimension table, 8 rows
│   └── fact_prices.csv         # fact table, 6,117 rows
│
└── README.md
```

---

## What I Learned

**Data Engineering**
- Multi-key lookup — matching on commodity AND date simultaneously; why a single-column join fails here
- Sort order matters for indexing — `iloc[0]` silently produces wrong results on descending data
- Unit conversion from raw API — cents/lb → $/kg is one line of code but requires reading API metadata carefully
- Star schema design — structuring data for Power BI consumption from the start saves rework later

**Statistics**
- Returns vs prices — raw prices create spurious correlation due to shared upward trend; `pct_change()` isolates genuine co-movement
- Lagged correlation — testing 0–6 month lags reveals whether markets react immediately or with delay
- Honest R² interpretation — 3.8% is not a failure; it reflects the reality of a diversified company

**Domain**
- Diversified FMCG companies are naturally hedged — no single commodity dominates the P&L
- Cost-side shocks (cocoa) hit stock price immediately; demand-side signals (coffee) lag by one month

---

## Limitations

- Yahoo Finance NESN.SW data starts from ~1990, limiting the analysis window despite commodity data going back to 1960
- Regression uses OLS without controlling for autocorrelation in residuals — `statsmodels` full summary recommended before drawing strong conclusions
- Alpha Vantage free tier excludes Cocoa, Palm Oil, and Soybeans

## Next Steps

- Add EUR/CHF FX rate and market index (S&P 500 or MSCI World) as control variables
- Test on quarterly frequency — may produce stronger signal as earnings reflect actual commodity costs
- Extend to peer comparison: Unilever (ULVR.L) or Mondelez (MDLZ)
- Build Power BI dashboard using exported CSV files

---

## Author

**Roman Honta** · Junior BI Analyst at Nestlé Ukraine  
[LinkedIn](#) · [Portfolio](#)
