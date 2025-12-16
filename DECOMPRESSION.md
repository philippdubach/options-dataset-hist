# Decompression

The SQLite databases are compressed with Zstandard (zstd). This document describes how to decompress them.

## Files

| Dataset | Compressed | Decompressed |
|---------|------------|--------------|
| SPY | 1.5 GB | 9 GB |
| IWM | 812 MB | 4.8 GB |
| QQQ | 965 MB | 5.6 GB |
| Total | 3.3 GB | 19.4 GB |

Ensure you have at least 20 GB of free disk space before decompressing all databases.

## Command Line

### macOS

```bash
brew install zstd
cd data/
zstd -d spy_options.db.zst
zstd -d iwm_options.db.zst
zstd -d qqq_options.db.zst
```

macOS 15 and later can decompress zst files natively in Finder.

### Linux

```bash
# Debian/Ubuntu
sudo apt-get install zstd

# Fedora/RHEL
sudo dnf install zstd

cd data/
zstd -d spy_options.db.zst
zstd -d iwm_options.db.zst
zstd -d qqq_options.db.zst
```

### Windows

```powershell
# Using Chocolatey
choco install zstandard

# Using Scoop
scoop install zstd

cd data
zstd -d spy_options.db.zst
zstd -d iwm_options.db.zst
zstd -d qqq_options.db.zst
```

7-Zip version 21.01 and later supports zst files through the GUI.

## Python

```python
import zstandard as zstd

def decompress(name):
    dctx = zstd.ZstdDecompressor()
    with open(f'data/{name}.zst', 'rb') as fin:
        with open(f'data/{name}', 'wb') as fout:
            dctx.copy_stream(fin, fout)

for db in ['spy_options.db', 'iwm_options.db', 'qqq_options.db']:
    decompress(db)
```

Install the zstandard package with `pip install zstandard`.

## Verification

```bash
sqlite3 data/spy_options.db "PRAGMA integrity_check;"
sqlite3 data/spy_options.db "SELECT COUNT(*) FROM options_data;"
```

Expected record counts: SPY 24.7M, IWM 13.4M, QQQ 15.3M.

## Alternative

The Parquet files in `data/parquet_*/` do not require decompression and can be loaded directly with pandas or polars. This format is suitable for most analytical workflows.
