# Systematic Multi-Asset Portfolio Research with Dynamic Black-Litterman Views

A research pipeline for systematic multi-asset allocation across 12 liquid ETFs. The project tests cross-sectional signals, converts validated rankings into relative Black-Litterman views, stabilizes covariance estimates with Ledoit-Wolf shrinkage, and evaluates constrained portfolios in a quarterly walk-forward backtest with transaction costs and turnover controls.

## Research objective

The goal is to study whether a transparent signal-to-portfolio process can improve on a neutral diversified allocation without treating historical backtest performance as proof of a standalone alpha model.

The workflow is:

1. Build a diversified ETF universe across equities, rates, credit and real assets.
2. Estimate rolling economic factor exposures for portfolio diagnostics.
3. Test six candidate cross-sectional signals using information coefficients, HAC inference and top-minus-bottom spreads.
4. Combine the strongest momentum and trend signals into a composite score.
5. Convert the score into relative Top-3 vs. Bottom-3 Black-Litterman views.
6. Estimate risk with a 60-month Ledoit-Wolf covariance matrix.
7. Optimize long-only portfolios subject to a 40% asset cap.
8. Compare Black-Litterman against Equal Weight, Minimum Variance, Historical Mean-Variance, Maximum Sharpe, Risk Parity and CVaR.
9. Evaluate performance, drawdown, tail risk, turnover, transaction costs, factor exposures and risk contribution in a walk-forward framework.

## Investable universe

| Sleeve | ETFs |
|---|---|
| US / international equity | SPY, QQQ, EFA, EEM |
| Rates / inflation | SHY, IEF, TIP |
| Credit | LQD, HYG |
| Real assets | GLD, DBC |
| Real estate | VNQ |

Diagnostic factor proxies are constructed for **Equity, Duration, Credit and Commodity** exposures using separate ETFs so that the factor set is not simply the investable universe relabeled.

## Signal research

Six candidate signals are tested:

- 12-1 momentum
- 6-1 momentum
- risk-adjusted momentum
- 10-month trend
- 12-month drawdown
- volatility-regime signal

The strongest individual signals were:

| Signal | Mean IC | HAC t-stat |
|---|---:|---:|
| 10-month trend | 0.090 | 2.68 |
| 12-1 momentum | 0.086 | 2.45 |
| 6-1 momentum | 0.080 | 2.25 |

The final composite gives equal weight to a momentum family and the trend signal. On one-month-ahead returns, the composite produced a **0.082 mean IC**, **2.53 HAC t-stat** and **0.011 HAC p-value** across 218 monthly observations.

Signal persistence is also examined before selecting the quarterly rebalance frequency rather than assuming that monthly trading is necessary.

## Black-Litterman view construction

At each rebalance date, assets are ranked by the composite signal. The three highest-ranked ETFs form the long side of a relative view and the three lowest-ranked ETFs form the short side.

The dimensionless score gap is translated into an expected relative return using a fixed annualized scaling parameter. Black-Litterman then combines that view with an equal-weight equilibrium prior and covariance-based view uncertainty.

Across the 57 quarterly walk-forward views:

- **68.4%** of Top-3 baskets outperformed the corresponding Bottom-3 baskets.
- Mean realized three-month Top-minus-Bottom spread was approximately **2.00%**.
- Mean expected three-month spread was approximately **1.40%**.
- Expected-versus-realized spread correlation was only about **0.17**.

This suggests the signal was more useful for the **direction** of relative performance than for precisely forecasting the magnitude of the spread.

## Covariance estimation

Portfolio risk is estimated using a rolling 60-month Ledoit-Wolf covariance matrix.

For the static diagnostic date, shrinkage reduced the covariance condition number from approximately **2,743 to 87**, materially improving numerical conditioning before optimization.

## Portfolio construction

The implementation is:

- long-only
- maximum asset weight: 40%
- quarterly rebalancing
- 60-month rolling estimation window
- 10 bps transaction-cost assumption
- turnover-aware Black-Litterman extension using an L1-style trading penalty

The optimizer comparison includes:

- Equal Weight
- Minimum Variance
- Historical Mean-Variance
- Maximum Sharpe
- Risk Parity
- Historical CVaR
- Black-Litterman
- Black-Litterman + Turnover Penalty

## Walk-forward results

The out-of-sample-style walk-forward return period runs from **May 2012 through July 2026**, with 57 non-overlapping quarterly rebalances.

| Strategy | Ann. Return | Sharpe | Max Drawdown | CVaR 95% | Ann. One-Way Turnover |
|---|---:|---:|---:|---:|---:|
| Equal Weight | 6.49% | 0.611 | -17.56% | -4.82% | 7.52% |
| Black-Litterman | 9.10% | 0.693 | -13.88% | -6.62% | 147.49% |
| BL + Turnover Penalty | 8.83% | 0.681 | -13.90% | -6.63% | 78.51% |

The plain Black-Litterman portfolio provides the cleanest test of whether the systematic views add value relative to the neutral benchmark. The turnover-aware version trades away roughly **27 bps of annualized return** while reducing annualized one-way turnover by about **47%**, making it the more practical implementation within this framework.

The project also reports Sortino ratio, volatility, transaction costs, effective number of holdings, realized factor exposures, factor-adjusted alpha and economic risk contribution by sleeve.

## Repository structure

- [PC_01_Data_Factors_and_Signal_Research.ipynb](./PC_01_Data_Factors_and_Signal_Research.ipynb)  
  Data preparation, factor diagnostics, candidate-signal construction, IC/HAC testing, signal persistence, economic spread validation and composite-signal construction.

- [PC_02_Black_Litterman_Portfolio_Construction_and_Backtest.ipynb](./PC_02_Black_Litterman_Portfolio_Construction_and_Backtest.ipynb)  
  Covariance estimation, Black-Litterman posterior, optimizer comparison, turnover-aware implementation, quarterly walk-forward backtest, factor attribution and risk diagnostics.

- [CQF_BL_Handoff.pkl](./CQF_BL_Handoff.pkl)  
  Serialized handoff from the signal-research notebook to the portfolio-construction notebook.

- [PC Lakshya Sharma REPORT.pdf](./PC%20Lakshya%20Sharma%20REPORT.pdf)  
  Written project report.

## Running the notebooks

Run the notebooks in order:

1. `PC_01_Data_Factors_and_Signal_Research.ipynb`
2. `PC_02_Black_Litterman_Portfolio_Construction_and_Backtest.ipynb`

The first notebook creates `CQF_BL_Handoff.pkl`, which is loaded by the second notebook.

Core Python packages used include:

`pandas`, `numpy`, `statsmodels`, `scikit-learn`, `scipy`, `matplotlib`, `seaborn`, `yfinance` and `pandas-datareader`.

## Important limitations

- Results are historical and depend on the selected ETF universe, signal definitions and implementation assumptions.
- Factor ETFs are diagnostic proxies rather than a commercial equity risk model.
- The mapping from signal magnitude to expected return is noisy; the project explicitly tests this calibration instability.
- Transaction costs are simplified and do not model market impact, bid-ask variation or capacity.
- Backtest results should be interpreted as research evidence, not as a live performance record.

## Author

**Lakshya Sharma**

Certificate in Quantitative Finance (CQF) project, 2026.
