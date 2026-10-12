# Project 2 — Portfolio Backtesting and Rolling Optimization with Python

This project investigates whether portfolios optimized using historical data continue to perform well on unseen future data.

It extends classical Markowitz portfolio optimization into an **out-of-sample backtesting framework** and compares static optimization, rolling re-optimization, equal-weight diversification, and a passive SPY benchmark.

The analysis uses four ETFs:

- SPY — S&P 500
- QQQ — Nasdaq-100
- TLT — Long-term U.S. Treasury bonds
- GLD — Gold

---

## Objective

A portfolio may appear optimal when evaluated on the same historical data used to construct it.

However,

$$
\text{in-sample optimal}
\neq
\text{out-of-sample optimal}
$$

The goal of this project is therefore to test portfolio strategies only on data that were not available when the portfolio weights were estimated.

The general workflow is:

$$
\text{Historical data}
\rightarrow
\text{Estimate parameters}
\rightarrow
\text{Optimize weights}
\rightarrow
\text{Future unseen data}
\rightarrow
\text{Evaluate performance}
$$

---

## Strategies Compared

Four strategies are evaluated.

### 1. Rolling Maximum-Sharpe Portfolio

At the beginning of each year, the optimizer uses only the previous five years of market data.

For example:

```text
2018–2022 -> optimize weights -> invest during 2023
2019–2023 -> optimize weights -> invest during 2024
2020–2024 -> optimize weights -> invest during 2025
