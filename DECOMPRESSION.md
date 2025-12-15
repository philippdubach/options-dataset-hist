# Decompression Instructions

The SQLite database is compressed using Zstandard (zstd) for efficient storage and fast decompression.

## Files

| Compressed File | Decompressed Output | Size |
|-----------------|---------------------|------|
| `data/spy_options.db.zst` | `data/spy_options.db` | 9 GB |

## Decompression Methods

### Command Line (zstd)

macOS (Homebrew):
```bash
brew install zstd
cd data/
zstd -d spy_options.db.zst
```

Linux (Debian/Ubuntu):
```bash
sudo apt-get install zstd
cd data/
zstd -d spy_options.db.zst
```

Linux (Fedora/RHEL):
```bash
sudo dnf install zstd
cd data/
zstd -d spy_options.db.zst
```

Windows (Chocolatey):
```powershell
choco install zstandard
cd data
zstd -d spy_options.db.zst
```

Windows (Scoop):
```powershell
scoop install zstd
cd data
zstd -d spy_options.db.zst
```

### Python

```python
import zstandard as zstd
import pathlib

compressed_path = pathlib.Path('data/spy_options.db.zst')
output_path = pathlib.Path('data/spy_options.db')

dctx = zstd.ZstdDecompressor()
with open(compressed_path, 'rb') as ifh:
    with open(output_path, 'wb') as ofh:
        dctx.copy_stream(ifh, ofh)

print(f"Decompressed to {output_path}")
```

Install the package first:
```bash
pip install zstandard
```

### 7-Zip (GUI)

1. Download and install [7-Zip](https://www.7-zip.org/) (version 21.01 or later)
2. Right-click on `spy_options.db.zst`
3. Select 7-Zip, then Extract Here

### macOS (Sequoia and later)

macOS 15+ has native zstd support. Double-click `spy_options.db.zst` in Finder to decompress.

## Verification

After decompression, verify file integrity:

```bash
# Check file size (should be approximately 9 GB)
ls -lh data/spy_options.db

# Verify database integrity
sqlite3 data/spy_options.db "PRAGMA integrity_check;"

# Quick sanity check
sqlite3 data/spy_options.db "SELECT COUNT(*) FROM options_data;"
# Expected: 24,681,665
```

### SHA256 Checksums

```bash
# Compressed file
sha256sum data/spy_options.db.zst
# cc66ac015dfc196c3621f4cc76c9a538835a213ee082b2ce95f1201964409efb

# Decompressed file
sha256sum data/spy_options.db
# 933b4c9cfe1e05fbb6ae14c22862dbb18e13a9c4d7bbb3b9beb2f3fbf1fd58ee
```

## Quick Start After Decompression

### Python
```python
import sqlite3
conn = sqlite3.connect('data/spy_options.db')
cursor = conn.cursor()
cursor.execute("SELECT COUNT(*) FROM options_data")
print(f"Total records: {cursor.fetchone()[0]:,}")
conn.close()
```

### Command Line
```bash
sqlite3 data/spy_options.db "SELECT date, COUNT(*) FROM options_data GROUP BY date ORDER BY date DESC LIMIT 5;"
```

### DuckDB with Parquet (no decompression needed)
```python
import duckdb
result = duckdb.query("SELECT COUNT(*) FROM 'data/parquet/*.parquet'")
print(result)
```

## Notes

- Ensure at least 10 GB of free disk space before decompressing
- Decompression typically takes 1-3 minutes depending on hardware
- The compressed file can be deleted after decompression to save space
- For columnar access, use the parquet files in `data/parquet/` which require no decompression
