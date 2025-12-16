# Historic Options Dataset

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Data Format](https://img.shields.io/badge/Format-SQLite%20%7C%20Parquet-blue.svg)](#data-formats)
[![Records](https://img.shields.io/badge/Records-53.4M-green.svg)](#dataset-overview)


This repository contains historical options data for three major U.S. equity ETFs: SPY (S&P 500), IWM (Russell 2000), and QQQ (Nasdaq-100). The dataset spans January 2008 to December 2025 and includes over 53 million option contracts with Greeks, implied volatilities, and market microstructure variables.

## Overview

| Symbol | Description | Records | Period | Compressed |
|--------|-------------|---------|--------|------------|
| SPY | SPDR S&P 500 ETF Trust | 24.7M | 2008-2025 | 1.5 GB |
| IWM | iShares Russell 2000 ETF | 13.4M | 2008-2025 | 812 MB |
| QQQ | Invesco QQQ Trust | 15.3M | 2011-2025 | 965 MB |
| Total | | 53.4M | | 3.3 GB |

The dataset covers several market regimes including the 2008 financial crisis, the 2020 COVID-19 volatility spike, and the subsequent recovery periods.

## Data Format

Data is provided in two formats. SQLite databases are compressed with Zstandard and require decompression before use (see [DECOMPRESSION.md](DECOMPRESSION.md)). Parquet files are partitioned by year and can be used directly.

### SQLite

| File | Compressed | Decompressed |
|------|------------|--------------|
| `data/spy_options.db.zst` | 1.5 GB | 9 GB |
| `data/iwm_options.db.zst` | 812 MB | 4.8 GB |
| `data/qqq_options.db.zst` | 965 MB | 5.6 GB |

### Parquet

| Directory | Size |
|-----------|------|
| `data/parquet_spy/` | 559 MB |
| `data/parquet_iwm/` | 313 MB |
| `data/parquet_qqq/` | 638 MB |

## Schema

The options data table contains the following columns:

| Column | Type | Description |
|--------|------|-------------|
| contract_id | TEXT | Unique contract identifier |
| symbol | TEXT | Underlying symbol |
| expiration | TEXT | Expiration date |
| strike | REAL | Strike price |
| type | TEXT | Call or put |
| bid, ask | REAL | Best bid and ask prices |
| volume | INTEGER | Daily trading volume |
| open_interest | INTEGER | Open interest |
| date | TEXT | Observation date |
| implied_volatility | REAL | Black-Scholes implied volatility |
| delta, gamma, theta, vega, rho | REAL | Option Greeks |

Each database also includes an underlying_prices table with daily OHLCV data.

## Usage

### Parquet

```python
import pandas as pd

# Load single year
df = pd.read_parquet('data/parquet_spy/options_2024.parquet')

# Load all years
from glob import glob
files = sorted(glob('data/parquet_spy/options_*.parquet'))
df = pd.concat([pd.read_parquet(f) for f in files])
```

### SQLite

```python
import sqlite3
import pandas as pd

conn = sqlite3.connect('data/spy_options.db')
df = pd.read_sql("SELECT * FROM options_data WHERE date >= '2024-01-01'", conn)
```

## Documentation

For detailed statistics and methodology, see the technical report in [reports/dataset_description.pdf](reports/dataset_description.pdf).

## Citation

```bibtex
@misc{dubach2025options,
  author = {Dubach, Philipp},
  title = {Historic Options Dataset: SPY, IWM, and QQQ Options 2008-2025},
  year = {2025},
  publisher = {GitHub},
  url = {https://github.com/philippdubach/historic-options-dataset}
}
```

## License

MIT License. See [LICENSE](LICENSE) for details.
