# Canadian Economic Stagflation Risk Assessment

Is Canada at risk of **stagflation**, meaning high inflation, high interest rates and stagnating growth all at once? This project uses the PESTEL framework to combine 15 Statistics Canada datasets into one monthly dataset and examines how Bank of Canada policy relates to growth, prices and unemployment.

**Tools:** Python · pandas · NumPy · statsmodels · Matplotlib · seaborn

> **Update:** the first version concluded that rate hikes had a "statistically significant, delayed impact" on unemployment. That p-value was overstated by autocorrelation in the monthly data. The regression was re-done with robust errors and a model of changes. See the [pull request](../../pulls?q=is%3Apr) for the full before/after.

## Business question

Do the Bank of Canada's interest-rate decisions show a measurable, delayed effect on the wider economy, and do recent indicators show the pattern of stagflation (rising rates and prices while growth stalls)?

## Data

All data comes from the [Statistics Canada Open Data Portal](https://www150.statcan.gc.ca/n1/en/type/data). The 15 source tables are grouped by PESTEL dimension:

| PESTEL dimension | Indicators |
|---|---|
| **Political** | Bank of Canada target rate, government bond yields, mortgage interest cost index |
| **Economic** | Real GDP by industry, exchange rates (USD, EUR, CERI), merchandise trade balance |
| **Social** | Retail beef prices, retail sales, unemployment and employment rates |
| **Technological** | Aircraft movements, rail freight, wholesale trade |
| **Environmental** | Softwood lumber production, crude oil production, electricity generation |

`Final_Dataset.csv` is the merged result: **926 monthly rows (1949–2026) × 44 indicators**.

### Data preparation

- **Mixed frequencies made monthly.** Daily series such as interest rates and exchange rates were averaged per month. Weekly counts such as aircraft movements were summed per month.
- **Signal over noise.** We used the official *Target Rate* (the policy lever) rather than market overnight rates, a single standard product (ground beef, $/kg) for food prices, and **seasonally adjusted** GDP and retail sales.
- **Full outer join on Year + Month.** Series start in different years (rates in 1949, modern GDP in 1997), so an outer join keeps all available history instead of cutting everything back to the shortest series.

## Analysis

1. **Trends:** target rate, real GDP, unemployment, lumber, oil, aircraft and rail traffic over time.
2. **Correlation:** cross-PESTEL correlations in levels and in **year-over-year changes**, which avoids spurious correlation between trending series.
3. **Regression:** unemployment vs. the target rate 3 months earlier, with a **Durbin-Watson** autocorrelation check, **Newey-West (HAC)** robust errors, and a model of 12-month changes at 3-, 6- and 12-month lags.
4. **Pattern analysis:** normalized rates, GDP, food prices and unemployment over 2015–2025.

## Key findings

- **The ingredients of stagflation risk appeared from 2022.** The average target rate rose from 0.25% (2021) to about 4.7% (2023–24), real GDP growth slowed from about 4.7% (2022) to about 2% and then 1.3% (2025), and ground beef prices rose 7–17% per year.
- **No evidence that rate hikes raised unemployment within a year.**

| Model | Coefficient | p-value |
|---|---|---|
| Original OLS, levels, 3-month lag | −0.18 | < 0.001 ❌ overstated |
| Same model, Newey-West errors | −0.18 | 0.081 |
| 12-month changes, 3-month lag | −0.25 | 0.015 |
| 12-month changes, 6-month lag | −0.10 | 0.307 |
| 12-month changes, 12-month lag | +0.13 | 0.223 |

The only significant relationship is **negative**: rates tend to rise when unemployment is falling, which is consistent with the Bank of Canada tightening in strong labour markets. Rates explain under 8% of the variation in unemployment in every model.

## How to run

```bash
pip install pandas numpy matplotlib seaborn statsmodels scikit-learn
jupyter notebook stagflation_analysis.ipynb
```

Keep `Final_Dataset.csv` in the same folder as the notebook.

## My contributions

- **Data cleaning:** cleaned and prepared the merged dataset for analysis.
- **Correlation analysis:** built the cross-PESTEL correlation matrix with the lagged policy rate.
- **Regression:** built and ran the lagged OLS regression model.

## Team

Group project for DAMO-511 Data Analytics Case Study 2, Master of Data Analytics, University of Niagara Falls Canada (2026).

Raul Alfonso Marroquin Puig · Samuel Ikemefuna Mbadiwe · Thirumurugan Nandhakumar Kumaran · **Lisandro Rios**
