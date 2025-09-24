# 📚 Cryptobooks  
**Final Year Project** – A lightweight Python library for encrypting, storing, and retrieving digital books securely.

---

## Table of Contents
1. [Overview](#overview)  
2. [Installation](#installation)  
3. [Quick Start](#quick-start)  
4. [Usage](#usage)  
   - [Command‑Line Interface (CLI)](#cli)  
   - [Python API](#python-api)  
5. [API Documentation](#api-documentation)  
   - [Modules & Classes](#modules--classes)  
   - [Key Functions & Methods](#key-functions--methods)  
6. [Examples](#examples)  
   - [Encrypt & Store a Book](#example‑encrypt‑store)  
   - [Search & Decrypt](#example‑search‑decrypt)  
   - [Batch Processing](#example‑batch)  
7. [Configuration](#configuration)  
8. [Testing](#testing)  
9. [Contributing](#contributing)  
10. [License](#license)  

---

## Overview <a name="overview"></a>

**Cryptobooks** is a small, self‑contained Python package designed for the final‑year project *“Secure Digital Library”*. It provides:

* **AES‑256 encryption** of e‑books (PDF, EPUB, MOBI, TXT).  
* **Metadata indexing** (title, author, tags) stored in an encrypted SQLite database.  
* **Command‑line tools** for everyday operations (add, list, retrieve, delete).  
* A **clean Python API** for integration into larger applications or research pipelines.  

> **Why Cryptobooks?**  
> • No external services – everything runs locally.  
> • Strong cryptography (AES‑GCM, PBKDF2‑HMAC‑SHA256).  
> • Simple, well‑documented API that can be extended for future features (e.g., DRM, cloud sync).

---

## Installation <a name="installation"></a>

### Prerequisites
| Requirement | Minimum Version |
|-------------|-----------------|
| Python      | 3.9+            |
| pip         | 21.0+           |
| SQLite      | Built‑in (Python stdlib) |

> **Note:** Cryptobooks works on Windows, macOS, and Linux.

### Install from PyPI (recommended)

```bash
pip install cryptobooks
```

### Install from source (development)

```bash
# Clone the repository
git clone https://github.com/yourusername/cryptobooks.git
cd cryptobooks

# Create a virtual environment (optional but recommended)
python -m venv .venv
source .venv/bin/activate   # .venv\Scripts\activate on Windows

# Install the package in editable mode with dev dependencies
pip install -e .[dev]
```

#### Optional extras
* `dev` – testing tools (`pytest`, `tox`), linting (`ruff`, `black`), and documentation (`mkdocs`).
* `cli` – installs the optional `cryptobooks-cli` entry‑point (included by default).

```bash
pip install cryptobooks[dev,cli]
```

---

## Quick Start <a name="quick-start"></a>

```bash
# Initialise a new encrypted library (creates `library.db` and a key file)
cryptobooks init --library ./my_library

# Add a book (will be encrypted automatically)
cryptobooks add ./books/Quantum_Computing.pdf --title "Quantum Computing" --author "N. Nielsen"

# List all books (metadata only, never reveals plaintext)
cryptobooks list

# Retrieve a book (decrypts to the specified output folder)
cryptobooks get "Quantum Computing" --out ./decrypted
```

All commands prompt for the master password (or you can provide it via `--password` for scripting).

---

## Usage <a name="usage"></a>

### Command‑Line Interface (CLI) <a name="cli"></a>

| Command | Description | Example |
|---------|-------------|---------|
| `cryptobooks init` | Initialise a new encrypted library. | `cryptobooks init --library ./my_library` |
| `cryptobooks add <file>` | Encrypt and add a book to the library. | `cryptobooks add book.epub --title "My Book"` |
| `cryptobooks list` | Show a table of stored books (metadata only). | `cryptobooks list --json` |
| `cryptobooks get <title>` | Decrypt a book to a folder. | `cryptobooks get "My Book" --out ./out` |
| `cryptobooks delete <title>` | Securely remove a book and its metadata. | `cryptobooks delete "My Book"` |
| `cryptobooks export-db` | Export the encrypted SQLite DB (for backup). | `cryptobooks export-db --out backup.db.enc` |
| `cryptobooks import-db` | Import a previously exported DB. | `cryptobooks import-db backup.db.enc` |

All commands accept `--password <pwd>` (or `-p`) to avoid interactive prompts, **but be careful** – avoid exposing passwords in shell history.

### Python API <a name="python-api"></a>

```python
from cryptobooks import CryptoLibrary, BookMeta

# Open (or create) a library
lib = CryptoLibrary(path="./my_library", password="my‑strong‑pwd")

# Add a book
meta = BookMeta(
    title="Deep Learning",
    author="Ian Goodfellow",
    tags=["AI", "Neural Networks"]
)
lib.add_book("./books/deep_learning.pdf", meta)

# Search
results = lib.search(title="deep learning")
print("Found:", results)

# Retrieve (decrypt) a book
output_path = lib.get_book("Deep Learning", out_dir="./decrypted")
print(f"Decrypted to {output_path}")

# Delete a book
lib.delete_book("Deep Learning")
```

> **Tip:** The `CryptoLibrary` class is a context manager, so you can safely use `with` blocks:

```python
with CryptoLibrary("./my_library", password="pwd") as lib:
    lib.add_book(...)
```

---

## API Documentation <a name="api-documentation"></a>

### Modules & Classes <a name="modules--classes"></a>

| Module | Class / Function | Purpose |
|--------|------------------|---------|
| `cryptobooks.core` | `CryptoLibrary` | Main entry point – handles encryption, DB access, and high‑level operations. |
| `cryptobooks.core` | `BookMeta` (dataclass) | Holds metadata (title, author, tags, added_at, checksum). |
| `cryptobooks.crypto` | `encrypt_file`, `decrypt_file` | Low‑level AES‑GCM helpers. |
| `cryptobooks.db` | `EncryptedDB` | Thin wrapper around SQLite that encrypts/decrypts rows on the fly. |
| `cryptobooks.cli` | `main` | Click‑based command‑line interface. |
| `cryptobooks.utils` | `hash_file`, `derive_key` | Utility functions for hashing, key derivation (PBKDF2). |
| `cryptobooks.exceptions` | `CryptoBooksError`, `AuthenticationError` | Custom exception hierarchy. |

### Key Functions & Methods <a name="key-functions--methods"></a>

#### `CryptoLibrary`

| Method | Signature | Description |
|--------|-----------|-------------|
| `__init__(self, path: str, password: str, *, iterations: int = 200_000)` | `CryptoLibrary(path, password, iterations=200_000)` | Opens/creates a library; derives the master key using PBKDF2‑SHA256. |
| `add_book(self, file_path: str, meta: BookMeta) -> None` | `add_book(file_path, meta)` | Encrypts `file_path` and stores metadata in the DB. |
| `search(self, *, title: str = None, author: str = None, tags: list[str] = None) -> list[BookMeta]` | `search(title=None, author=None, tags=None)` | Returns matching metadata objects. |
| `get_book(self, title: str, out_dir: str) -> str` | `get_book(title, out_dir)` | Decrypts the book with the given title to `out_dir`; returns the absolute path of the decrypted file. |
| `delete_book(self, title: str) -> None` | `delete_book(title)` | Securely wipes the encrypted blob and removes metadata. |
| `export_db(self, out_path: str) -> None` | `export_db(out_path)` | Writes the whole encrypted SQLite file to `out_path`. |
| `import_db(self, in_path: str) -> None` | `import_db(in_path)` | Replaces the current DB with the encrypted file at `in_path`. |
| `close(self) -> None` | `close()` | Closes DB connections; also called automatically on exit of a context manager. |

#### Low‑level crypto helpers (`cryptobooks.crypto`)

| Function | Signature | Notes |
|----------|-----------|-------|
