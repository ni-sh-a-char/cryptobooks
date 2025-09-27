# 📚 Cryptobooks  
**Final Year Project** – A Python library for managing, encrypting, and visualising personal finance data (books, ledgers, and crypto‑asset records).  

---  

## Table of Contents
1. [Overview](#overview)  
2. [Features](#features)  
3. [Installation](#installation)  
4. [Quick Start / Usage](#quick-start--usage)  
5. [API Documentation](#api-documentation)  
6. [Examples](#examples)  
7. [Configuration](#configuration)  
8. [Testing](#testing)  
9. [Contributing](#contributing)  
10. [License](#license)  

---  

## Overview
**Cryptobooks** is a lightweight, pure‑Python package designed for students and hobbyists who want to keep a secure, searchable record of their financial transactions, especially those involving cryptocurrencies.  
It provides:

* **Encrypted storage** – AES‑256 GCM encryption of all data files.  
* **Simple ledger model** – Accounts, categories, tags, and automatic balance calculations.  
* **Import/Export** – CSV, JSON, and SQLite back‑ends.  
* **Analytics & visualisation** – Built‑in helpers for time‑series plots, portfolio allocation charts, and tax‑report generation.  
* **CLI & Python API** – Use it from the command line or embed it in your own scripts.

---  

## Features
| Feature | Description |
|---------|-------------|
| **Secure storage** | All data is encrypted at rest with a user‑provided passphrase. |
| **Multiple back‑ends** | Choose between a single encrypted JSON file, an encrypted SQLite DB, or a cloud‑sync (S3) option. |
| **Extensible schema** | Custom fields, tags, and user‑defined transaction types. |
| **CLI** | `cryptobooks` command with sub‑commands (`add`, `list`, `balance`, `export`, …). |
| **Analytics** | Portfolio performance, ROI, tax‑lot tracking, and visualisations via Matplotlib/Plotly. |
| **Unit‑tested** | 95 % test coverage, CI on GitHub Actions. |
| **Documentation** | Full API reference generated with MkDocs + Material theme. |

---  

## Installation  

### Prerequisites
* Python **3.9** or newer  
* `pip` (>= 23.0)  
* Optional: `git` (for installing from source)  

### From PyPI (recommended)

```bash
# Create a virtual environment (optional but recommended)
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

# Install the package
pip install cryptobooks
```

### From source (latest development version)

```bash
# Clone the repository
git clone https://github.com/yourusername/cryptobooks.git
cd cryptobooks

# Install in editable mode with development dependencies
pip install -e .[dev]
```

### Optional dependencies
| Feature | Extra name | Packages installed |
|---------|------------|--------------------|
| Plotting & interactive charts | `plot` | `matplotlib`, `plotly`, `seaborn` |
| Cloud sync (AWS S3) | `cloud` | `boto3` |
| CSV/Excel import/export | `excel` | `pandas`, `openpyxl` |
| Testing & linting | `dev` | `pytest`, `pytest-cov`, `ruff`, `mypy` |

Install any extra with:

```bash
pip install cryptobooks[plot,excel]
```

---  

## Quick Start / Usage  

### 1️⃣ Initialise a new book

```bash
cryptobooks init mybook
# You will be prompted for a passphrase (used for encryption)
```

This creates an encrypted `mybook.cbk` file in the current directory.

### 2️⃣ Add a transaction (CLI)

```bash
cryptobooks add \
    --date 2024-09-01 \
    --account "Bank:Checking" \
    --category "Salary" \
    --amount 2500 \
    --currency USD \
    --description "September salary"
```

### 3️⃣ List recent transactions

```bash
cryptobooks list --limit 10
```

### 4️⃣ Show balances per account

```bash
cryptobooks balance
```

### 5️⃣ Export to CSV (for external analysis)

```bash
cryptobooks export --format csv --output september.csv
```

### 6️⃣ Use the library in Python code

```python
from cryptobooks import CryptoBook, Transaction

# Open an existing book (will ask for the passphrase)
book = CryptoBook.open("mybook.cbk")

# Add a transaction programmatically
tx = Transaction(
    date="2024-09-15",
    account="Crypto:Binance",
    category="Trade",
    amount=-0.015,
    currency="BTC",
    description="Bought BTC with USD"
)
book.add_transaction(tx)

# Persist changes
book.save()
```

---  

## API Documentation  

Below is a concise reference for the most important classes and functions. Full docstrings and type hints are available in the generated MkDocs site (`docs/`).

### `cryptobooks.core.CryptoBook`
| Method | Signature | Description |
|--------|-----------|-------------|
| `open(path: str, passphrase: str \| None = None) -> CryptoBook` | Opens an existing encrypted book. If `passphrase` is `None`, the user is prompted. |
| `create(path: str, passphrase: str) -> CryptoBook` | Creates a new book file. |
| `add_transaction(tx: Transaction) -> None` | Append a transaction to the ledger. |
| `list_transactions(filter: TransactionFilter \| None = None) -> List[Transaction]` | Return transactions matching the filter. |
| `balance(account: str \| None = None) -> Decimal` | Compute the current balance for a specific account or for all accounts. |
| `export(format: Literal["csv","json","sqlite"], **kwargs) -> bytes \| str` | Export the whole ledger. |
| `save() -> None` | Write changes back to the encrypted file. |
| `close() -> None` | Securely wipe in‑memory keys. |

### `cryptobooks.models.Transaction`
```python
class Transaction:
    date: str               # ISO‑8601 (YYYY‑MM‑DD)
    account: str            # e.g. "Bank:Checking" or "Crypto:Binance"
    category: str           # e.g. "Salary", "Trade", "Food"
    amount: Decimal         # Positive = inflow, negative = outflow
    currency: str           # ISO‑4217 code (USD, EUR, BTC, ETH, …)
    description: str = ""   # Optional free‑form text
    tags: List[str] = []    # Optional user tags
```

### `cryptobooks.utils`
| Function | Signature | Description |
|----------|-----------|-------------|
| `derive_key(passphrase: str, salt: bytes) -> bytes` | Derives a 256‑bit AES key using PBKDF2‑HMAC‑SHA256 (default 200 000 iterations). |
| `encrypt(data: bytes, key: bytes) -> bytes` | Returns `nonce || ciphertext || tag`. |
| `decrypt(blob: bytes, key: bytes) -> bytes` | Validates tag and returns plaintext. |
| `parse_date(s: str) -> datetime.date` | Accepts many common date formats. |
| `human_readable_amount(amount: Decimal, currency: str) -> str` | Formats with appropriate symbol and thousands separator. |

### Analytics helpers (`cryptobooks.analysis`)

| Function | Signature | Description |
|----------|-----------|-------------|
| `portfolio_performance(book: CryptoBook, start: str, end: str) -> pd.DataFrame` | Returns daily portfolio value (converted to a base currency). |
| `tax_lot_report(book: CryptoBook, year: int) -> pd.DataFrame` | Generates a FIFO/LIFO tax‑lot report for crypto disposals. |
| `plot_balance_over_time(book: CryptoBook, account: str \| None = None)` | Returns a Matplotlib `Figure`. |
| `plot_allocation_pie(book: CryptoBook, as_of: str)` | Returns a Plotly `Figure`. |

---  

## Examples  

### Example 1 – Building a simple personal finance dashboard  

```python
import pandas as pd
import matplotlib.pyplot as plt
from cryptobooks import CryptoBook, analysis

# Open the book (passphrase will be asked interactively)
book = CryptoBook.open("mybook.cbk")

# 1️⃣ Get a DataFrame of all transactions
df = pd.DataFrame([t.as_dict() for t in book.list_transactions()])

# 2️⃣ Compute monthly cash‑flow per category
monthly = (
    df.assign(date=pd.to_datetime(df["date"]))
      .set_index("date")
      .groupby([pd.Grouper(freq="M"), "category"])["amount"]
      .sum()
      .unstack(fill_value=0)
)

monthly.plot(kind="bar", stacked