# SPY Options Dataset (2008-2025)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Data Format](https://img.shields.io/badge/Format-SQLite%20%7C%20Parquet-blue.svg)](#data-formats)
[![Records](https://img.shields.io/badge/Records-24.7M-green.svg)](#dataset-overview)

A comprehensive dataset of SPY (SPDR S&P 500 ETF Trust) options spanning January 2008 to December 2025. The dataset contains approximately 24.7 million option contracts with associated Greeks, implied volatilities, and market microstructure variables.

## Dataset Overview

| Metric | Value |
|--------|-------|
| Total Records | 24,681,665 |
| Time Period | 2008-01-02 to 2025-12-12 |
| Trading Days | 4,514 |
| Years Covered | 17.9 |
| Calls | 12,340,771 |
| Puts | 12,340,894 |
| Avg Records/Day | 5,468 |

### Market Regimes Covered

- 2008-2009: Global Financial Crisis (VIX > 80)
- 2010-2019: Extended bull market with occasional volatility spikes
- 2020: COVID-19 pandemic volatility (VIX spike to 82)
- 2021-2025: Post-pandemic normalization and rate hiking cycle

## Data Formats

The dataset is available in two formats:

| Format | File | Size | Use Case |
|--------|------|------|----------|
| SQLite | `data/spy_options.db.zst` | 1.5 GB (compressed) | Full SQL queries, flexible analysis |
| Parquet | `data/parquet/` | 559 MB | Columnar analytics, ML pipelines |

The SQLite database requires decompression before use. See [DECOMPRESSION.md](DECOMPRESSION.md) for instructions. Parquet files can be queried directly without decompression.

## Schema

### Options Data Table

| Column | Type | Description |
|--------|------|-------------|
| `contract_id` | TEXT | Unique option contract identifier |
| `symbol` | TEXT | Underlying symbol (SPY) |
| `expiration` | TEXT | Contract expiration date (YYYY-MM-DD) |
| `strike` | REAL | Strike price in USD |
| `type` | TEXT | Option type ('call' or 'put') |
| `last` | REAL | Last traded price |
| `mark` | REAL | Mid-market price |
| `bid` | REAL | Best bid price |
| `bid_size` | INTEGER | Size at best bid |
| `ask` | REAL | Best ask price |
| `ask_size` | INTEGER | Size at best ask |
| `volume` | INTEGER | Daily trading volume |
| `open_interest` | INTEGER | Open interest |
| `date` | TEXT | Observation date (YYYY-MM-DD) |
| `implied_volatility` | REAL | Black-Scholes implied volatility |
| `delta` | REAL | First derivative w.r.t. underlying |
| `gamma` | REAL | Second derivative w.r.t. underlying |
| `theta` | REAL | Time decay per day |
| `vega` | REAL | Sensitivity to volatility |
| `rho` | REAL | Sensitivity to interest rate |
| `in_the_money` | INTEGER | ITM indicator (0 or 1) |

### Underlying Prices Table

Daily OHLCV data for SPY, enabling computation of moneyness and realized volatility measures.

## Quick Start

### Python with SQLite

```python
import sqlite3
import pandas as pd

conn = sqlite3.connect('data/spy_options.db')

df = pd.read_sql_query("""
    SELECT date, strike, type, implied_volatility, delta, volume
    FROM options_data
    WHERE date >= '2024-01-01'
    AND volume > 100
    LIMIT 10000
""", conn)

print(df.describe())
conn.close()
```

### Python with Parquet

```python
import pandas as pd

df = pd.read_parquet('data/parquet/',
                     filters=[('date', '>=', '2024-01-01')])
```

### DuckDB

```python
import duckdb

result = duckdb.query("""
    SELECT date, AVG(implied_volatility) as avg_iv
    FROM 'data/parquet/*.parquet'
    WHERE type = 'put'
    GROUP BY date
    ORDER BY date
""").df()
```

## Summary Statistics

### Implied Volatility by Option Type

| Type | Mean | Std | Min | Median | Max |
|------|------|-----|-----|--------|-----|
| Call | 0.220 | 0.127 | 0.001 | 0.183 | 9.99 |
| Put | 0.321 | 0.199 | 0.001 | 0.268 | 9.99 |
| Overall | 0.270 | 0.172 | 0.001 | 0.222 | 9.99 |

### Volume Statistics

| Type | Total Volume | Avg Daily Vol | Total OI | Avg OI |
|------|--------------|---------------|----------|--------|
| Call | 7.27B | 589 | 28.8B | 2,333 |
| Put | 9.99B | 810 | 55.8B | 4,518 |

### Data Quality

| Column | Available (%) |
|--------|---------------|
| implied_volatility | 99.09 |
| delta/gamma/theta/vega | 98.19 |
| volume | 100.00 |
| open_interest | 99.44 |

## Use Cases

This dataset supports research in:

- Implied volatility surface modeling and interpolation
- Machine learning approaches to option pricing
- Volatility forecasting and term structure analysis
- Market microstructure analysis of options markets
- Risk management and hedging strategy evaluation

### Feature Engineering

```python
import numpy as np

# Moneyness
df['moneyness'] = df['strike'] / df['spot_price']
df['log_moneyness'] = np.log(df['moneyness'])

# Time to expiration
df['days_to_expiry'] = (pd.to_datetime(df['expiration']) - pd.to_datetime(df['date'])).dt.days
df['sqrt_tau'] = np.sqrt(df['days_to_expiry'] / 365)

# Spread
df['spread'] = df['ask'] - df['bid']
df['spread_pct'] = df['spread'] / df['mark']
```

## Citation

If you use this dataset in your research, please cite:

```bibtex
@misc{dubach2025spy,
  author = {Dubach, Philipp},
  title = {SPY Options Dataset: A Foundation for Machine Learning-Based Volatility Modeling},
  year = {2025},
  publisher = {GitHub},
  url = {https://github.com/philippdubach/spy-options-dataset}
}
```

## Documentation

- [DECOMPRESSION.md](DECOMPRESSION.md): Extraction instructions for the compressed SQLite database
- [reports/dataset_description.pdf](reports/dataset_description.pdf): Full technical documentation

## Disclaimer

This dataset is provided for educational and research purposes only. Options trading involves significant risk. The data was sourced from a premium market data provider and processed for academic use. No warranty is made regarding the accuracy or completeness of the data.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
