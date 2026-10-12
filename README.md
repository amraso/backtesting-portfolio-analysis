# Project 2 — Portfolio Backtesting and Rolling Optimization with Python

This project investigates whether portfolios optimized using historical market data continue to perform well on **unseen future data**.

It extends classical Markowitz portfolio optimization into an **out-of-sample backtesting framework** and compares four strategies:

- Rolling maximum-Sharpe optimization
- Static maximum-Sharpe optimization
- Equal-weight portfolio
- SPY buy-and-hold benchmark

The analysis uses four ETFs:

- **SPY** — S&P 500 ETF
- **QQQ** — Nasdaq-100 ETF
- **TLT** — Long-term U.S. Treasury bond ETF
- **GLD** — Gold ETF

---

## Objective

A portfolio can appear highly attractive when evaluated using the same historical data that were used to construct it.

However,

$$
\text{in-sample optimal}
\neq
\text{out-of-sample optimal}
$$

The objective of this project is therefore to evaluate portfolio strategies using only information that would have been available at the time each investment decision was made.

The general workflow is:

$$
\text{Historical data}
\rightarrow
\text{Parameter estimation}
\rightarrow
\text{Portfolio optimization}
\rightarrow
\text{Future unseen data}
\rightarrow
\text{Performance evaluation}
$$

This structure helps avoid **look-ahead bias**.

---

# Strategies

## 1. Rolling Maximum-Sharpe Portfolio

At the beginning of each year, the strategy uses only the previous five years of market data.

For example:

```text
2018–2022 -> optimize weights -> invest during 2023
2019–2023 -> optimize weights -> invest during 2024
2020–2024 -> optimize weights -> invest during 2025
```

For each rolling training window:

1. Historical daily returns are calculated.
2. Mean returns are estimated.
3. The covariance matrix is estimated.
4. Maximum-Sharpe portfolio weights are calculated.
5. The portfolio is rebalanced at the beginning of the following year.
6. The resulting positions are held until the next annual rebalance.

The process is then repeated with a new five-year historical window.

---

## 2. Static Maximum-Sharpe Portfolio

The static strategy performs the optimization only once using historical data available before the selected out-of-sample period.

The resulting weights are then kept fixed throughout the backtest.

This allows comparison between:

$$
\text{static optimization}
$$

and

$$
\text{adaptive rolling optimization}.
$$

---

## 3. Equal-Weight Portfolio

The equal-weight portfolio allocates the same amount of capital to each ETF:

$$
w =
(0.25,\;0.25,\;0.25,\;0.25)
$$

corresponding to:

- 25% SPY
- 25% QQQ
- 25% TLT
- 25% GLD

The portfolio is rebalanced back to equal weights at the beginning of every year.

During the year, asset weights are allowed to drift naturally as prices change.

This provides a simple diversification benchmark that requires no return or covariance estimation.

---

## 4. SPY Buy-and-Hold

SPY is used as a passive U.S. equity benchmark.

An initial investment is made at the beginning of the backtest period and then held without rebalancing.

---

# Portfolio Theory

## Expected Portfolio Return

The expected portfolio return is

$$
\mu_p = w^T\mu
$$

where:

- $w$ is the vector of portfolio weights
- $\mu$ is the vector of expected asset returns

---

## Portfolio Variance

Portfolio variance is

$$
\sigma_p^2 = w^T\Sigma w
$$

where:

- $\Sigma$ is the covariance matrix of asset returns

Portfolio volatility is therefore

$$
\sigma_p =
\sqrt{w^T\Sigma w}.
$$

The covariance terms allow the model to account for diversification between assets.

---

# Sharpe Ratio

Portfolio weights are selected by maximizing the Sharpe ratio:

$$
S =
\frac{\mu_p-r_f}{\sigma_p}
$$

where:

- $\mu_p$ is expected portfolio return
- $r_f$ is the risk-free rate
- $\sigma_p$ is portfolio volatility

The Sharpe ratio measures **excess expected return per unit of volatility**.

The numerical optimizer uses SciPy's **Sequential Least Squares Programming (SLSQP)** algorithm.

Because `scipy.optimize.minimize()` performs minimization, the code minimizes

$$
-S
$$

which is equivalent to maximizing

$$
S.
$$

In the current implementation,

$$
r_f = 0
$$

is used as a simplifying educational assumption.

A future version can replace this with a realistic short-term risk-free benchmark.

---

# Optimization Constraints

All optimized portfolios satisfy:

$$
\sum_i w_i = 1
$$

which ensures that the portfolio is fully invested.

Short selling is not permitted:

$$
0 \leq w_i \leq 1.
$$

---

# Rolling Rebalancing

The rolling strategy uses a five-year lookback window.

At every annual rebalance:

$$
\text{past 5 years}
\rightarrow
\mu,\Sigma
\rightarrow
\text{maximum-Sharpe weights}
\rightarrow
\text{hold for next year}
$$

The important information flow is therefore:

$$
\boxed{
\text{past information}
\rightarrow
\text{portfolio decision}
\rightarrow
\text{future performance}
}
$$

and never:

$$
\text{future information}
\rightarrow
\text{past decision}.
$$

This is essential for a meaningful backtest.

---

# Buy-and-Hold Between Rebalances

After weights are selected, capital is allocated among the ETFs at the beginning of each year.

The corresponding number of shares is calculated as

$$
N_i =
\frac{w_i V_0}{P_{i,0}}
$$

where:

- $N_i$ is the number of shares of asset $i$
- $w_i$ is its portfolio weight
- $V_0$ is portfolio value at the beginning of the period
- $P_{i,0}$ is the asset price at the beginning of the period

These shares are then held throughout the year.

The portfolio is rebalanced only when the next annual period begins.

---

# Performance Metrics

Each strategy is evaluated using several risk and return measures.

## Annualized Return

Daily arithmetic returns are annualized as

$$
\mu_{\text{annual}}
=
252\,E[r_d].
$$

---

## Annualized Volatility

Daily volatility is annualized as

$$
\sigma_{\text{annual}}
=
\sqrt{252}\,\sigma_d.
$$

---

## Sharpe Ratio

$$
S =
\frac{\mu-r_f}{\sigma}.
$$

---

## Total Return

$$
R_{\text{total}}
=
\frac{V_{\text{final}}}
{V_{\text{initial}}}
-1.
$$

---

## Drawdown

At any time $t$, drawdown is defined as

$$
D_t =
\frac{V_t}
{\max_{\tau\leq t}V_\tau}
-1.
$$

Maximum drawdown is the largest observed loss from a previous portfolio peak.

---

# Example Results

For the long historical backtest used during development, the strategies produced approximately:

| Strategy | Annual Return | Volatility | Sharpe | Total Return | Max Drawdown |
|---|---:|---:|---:|---:|---:|
| Rolling Optimized | 10.92% | 11.76% | 0.929 | 647.10% | -30.02% |
| Static Optimized | 12.11% | 12.56% | 0.964 | 825.22% | -32.99% |
| Equal Weight | 10.69% | 10.98% | 0.973 | 625.85% | -25.03% |
| SPY | 12.42% | 19.57% | 0.635 | 689.13% | -55.19% |

Results depend on the selected historical window and on the latest market data available when the script is executed.

---

# Interpretation

Several useful observations emerge from the backtest.

### SPY produced strong absolute returns

SPY achieved a high annual return, but this came with considerably higher volatility and a much larger historical drawdown.

### Diversification reduced portfolio risk

Both optimized and equal-weight portfolios substantially reduced volatility and drawdowns compared with a pure-equity SPY allocation.

### Equal weighting remained highly competitive

The simple equal-weight strategy achieved strong risk-adjusted performance despite requiring no estimation of expected returns or covariance.

This illustrates an important quantitative-finance principle:

$$
\boxed{
\text{more complex}
\neq
\text{automatically better}
}
$$

### Optimized weights are unstable

The rolling maximum-Sharpe allocation can change substantially from year to year.

This reflects the sensitivity of mean-variance optimization to estimates of:

$$
\mu
$$

and

$$
\Sigma.
$$

Historical optimal allocations are therefore not necessarily stable through time.

---

# Figures

## Out-of-Sample Portfolio Performance

![Out-of-sample portfolio backtest](figures/out_of_sample_backtest.png)

The figure compares the growth of an initial portfolio value of 100 for:

- Rolling optimized portfolio
- Static optimized portfolio
- Equal-weight portfolio
- SPY buy-and-hold

---

## Rolling Maximum-Sharpe Weights

![Rolling optimized weights](figures/rolling_weights_lines.png)

This figure shows how the optimized allocation changes from one annual rebalance to the next.

Large changes in portfolio weights illustrate the sensitivity of maximum-Sharpe optimization to the historical estimation window.

---

## Rolling Portfolio Allocation — Stacked View

![Rolling weights stacked](figures/rolling_weights_stacked.png)

The stacked-area representation shows how the complete 100% portfolio allocation changes through time.

At every rebalance:

$$
\sum_i w_i = 1.
$$

---

# Generated Results

The script automatically generates numerical outputs in:

```text
results/
├── performance_comparison.csv
├── rolling_optimized_weights.csv
└── portfolio_values.csv
```

### `performance_comparison.csv`

Contains:

- annual return
- volatility
- Sharpe ratio
- total return
- maximum drawdown
- final portfolio value

for each strategy.

### `rolling_optimized_weights.csv`

Contains the optimized ETF allocation for every annual rebalancing year.

### `portfolio_values.csv`

Contains the complete historical portfolio-value series for every strategy.

---

# Repository Structure

```text
backtesting-portfolio-analysis/
│
├── backtesting_portfolio_analysis.py
├── README.md
├── requirements.txt
├── .gitignore
│
├── figures/
│   ├── out_of_sample_backtest.png
│   ├── rolling_weights_lines.png
│   └── rolling_weights_stacked.png
│
└── results/
    ├── performance_comparison.csv
    ├── rolling_optimized_weights.csv
    └── portfolio_values.csv
```

---

# Technologies

- Python
- NumPy
- pandas
- Matplotlib
- SciPy
- yfinance

---

# Installation

Clone the repository:

```bash
git clone https://github.com/amraso/backtesting-portfolio-analysis.git
```

Enter the project directory:

```bash
cd backtesting-portfolio-analysis
```

Create a virtual environment:

```bash
python3 -m venv .venv
```

Activate it:

```bash
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Run the analysis:

```bash
python backtesting_portfolio_analysis.py
```

---

# Requirements

`requirements.txt`:

```text
numpy
pandas
matplotlib
scipy
yfinance
```

---

# Limitations

This project is an educational implementation of portfolio backtesting and should not be interpreted as investment advice.

Important limitations include:

- expected returns are estimated from historical data
- covariance relationships may change over time
- optimization is sensitive to estimation error
- transaction costs are ignored
- bid-ask spreads are ignored
- taxes are ignored
- market impact is ignored
- annual rebalancing is assumed to occur without cost
- the risk-free rate is currently fixed at zero
- only four ETFs are considered
- short selling is not permitted
- past performance does not imply future performance

---

# Possible Extensions

Future versions could investigate:

- realistic time-dependent risk-free rates
- transaction costs
- quarterly or monthly rebalancing
- alternative rolling-window lengths
- additional asset classes
- minimum-variance portfolios
- maximum-diversification portfolios
- Sortino ratio
- Value at Risk (VaR)
- Expected Shortfall
- volatility targeting
- robust covariance estimators
- shrinkage methods
- Black-Litterman optimization
- comparison with additional benchmarks

---

## Disclaimer

This repository is intended for educational and quantitative-finance research purposes only.

It does not constitute financial or investment advice.
