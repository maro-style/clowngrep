# ClownGrep

ClownGrep is a local Python utility for matching domains associated with customers and third parties inside text, data-leak-style and modern Excel datasets.

It reads a local `clienti.csv`, recursively scans supported files under `file/`, and writes matched content into separate customer/third-party output folders plus a per-input report.

> **Privacy note:** real customer mappings, raw datasets and generated outputs may contain sensitive information. The repository is configured so that the operational `clienti.csv`, the contents of `file/`, and `outputs/` are ignored by Git.

## Features

- Recursive scanning under `file/`
- Customer and third-party domain mapping from CSV
- Exact-domain and subdomain-to-owner matching
- `RIGHE / dataleak` mode for line-oriented data
- `BLOCCHI` mode for blank-line-delimited multi-line TXT/LOG records
- `SCEGLI PER FILE` mode for mixed TXT/LOG structures
- Excel processing for `.xlsx` and `.xlsm`
- Automatic local `openpyxl` setup when required
- `ripgrep` support with `grep` fallback
- Per-customer and per-third-party outputs
- Per-input reports
- Streaming-oriented processing for large datasets
- Validation of domains and conflicting owner mappings
- Temporary-file cleanup and safer output-path handling
- Built-in terminal help and a detailed Italian usage guide

## Supported formats

`txt`, `log`, `csv`, `sql`, `json`, `xlsx`, `xlsm`

Legacy `.xls` files are not supported. Convert them to `.xlsx` before processing.

## Requirements

- Python 3.9+
- `ripgrep` (`rg`) recommended for performance, or `grep` as fallback
- `openpyxl` only when Excel files are processed

On macOS, `ripgrep` can be installed with Homebrew:

```bash
brew install ripgrep
```

You normally do **not** need to install `openpyxl` manually. If `.xlsx` or `.xlsm` files are detected and the dependency is missing, ClownGrep creates a private `.clowngrep_venv`, installs the project requirements there and restarts itself. This avoids modifying Homebrew/system Python installations protected by PEP 668.

Manual dependency installation remains possible:

```bash
python3 -m pip install -r requirements.txt
```

## Quick start

Create the operational customer mapping from the safe template:

```bash
cp clienti.example.csv clienti.csv
```

Edit `clienti.csv`, then place the files to scan under:

```text
file/
```

Run:

```bash
python3 clowngrep.py
```

### Customer CSV format

```text
third parties of,customer,value
,Example Corp,example.com
Example Corp,Example SaaS,example-saas.com
```

Interpretation:

- empty `third parties of`: `customer` is the direct customer that owns the domain;
- non-empty `third parties of`: this column contains the parent customer, while `customer` contains the third-party organization name;
- `value`: domain to search for. Leading `@` or `.` characters are normalized away.

If the same domain is assigned to different owners, ClownGrep stops instead of silently selecting one mapping.

## Processing modes

For each detected extension, ClownGrep asks how it should be processed.

### 1 — RIGHE / dataleak

Each line is an independent record. Only lines containing a target domain are saved.

Use it for CSV, SQL dumps, JSON/JSONL, line-oriented logs, credential lists and TXT files where one record equals one line.

### 2 — BLOCCHI

Available only for `.txt` and `.log`. Multiple consecutive lines form one record and blank lines separate records. If any line in a block contains a target domain, the **whole block** is saved.

Use it for multi-line records such as:

```text
URL: portal.example.com
User: mario
Password: example-password
Browser: Chrome

URL: unrelated.example
User: luca
Password: another-example
```

**Important:** BLOCCHI depends on blank-line separators. Without them, a whole file may be interpreted as one block.

### 3 — SCEGLI PER FILE

Available only for `.txt` and `.log`. Use this when files with the same extension have different structures. ClownGrep asks whether to use RIGHE or BLOCCHI for each file and also allows a single file to be skipped.

### Help

At any mode prompt, type:

```text
h
```

for examples and a quick explanation.

Practical rule:

```text
one record per line                     -> 1
one record across multiple lines        -> 2
same extension, mixed file structures   -> 3
```

See [`GUIDA_USO.md`](GUIDA_USO.md) for detailed examples.

## Output

Generated data is written under `outputs/`:

```text
outputs/
├── clienti/
│   └── Example_Corp/
├── terze_parti/
│   └── Example_Corp/
│       └── Example_SaaS/
└── output_<source>.txt
```

The reports contain the source file, matched domain and matched content. Treat them as potentially sensitive.

## Safe example dataset

A complete synthetic test pack is included in `examples/`:

```text
examples/
├── clienti.csv
├── TEST_INSTRUCTIONS.md
├── EXPECTED_MATCHES.md
└── file/
    ├── 01_dataleak.txt
    ├── 02_blocks.log
    ├── 03_records.csv
    ├── 04_dump.sql
    ├── 05_records.json
    ├── 07_workbook.xlsx
    ├── 08_workbook.xlsm
    └── nested/
        └── 06_nested.txt
```

All organizations, domains and records are fictional. The examples use the reserved `.example` namespace.

To run the example set without exposing real data:

```bash
cp examples/clienti.csv clienti.csv
cp -R examples/file/. file/
python3 clowngrep.py
```

Suggested modes are documented in `examples/TEST_INSTRUCTIONS.md`.

## Privacy and repository safety

The default `.gitignore` excludes:

- operational `clienti.csv`
- everything placed under root `file/`
- `outputs/`
- `.clowngrep_venv/`
- Python caches and common editor files

The synthetic files under `examples/` are intentionally versioned and safe for public testing.

Before every commit, verify:

```bash
git status
```

Do not publish real customer mappings, raw leak material, credentials, PII, API keys or internal data in commits, issues or pull requests.

## Security / scope

ClownGrep is a local triage utility. It does not download data, authenticate to external services or perform exploitation. Use it only on data you are authorized to access and process.

See [`SECURITY.md`](SECURITY.md) for responsible-reporting guidance.

## License

ClownGrep is distributed under the **PolyForm Noncommercial License 1.0.0** (`PolyForm-Noncommercial-1.0.0`).

Noncommercial use, modification and distribution are permitted subject to the license terms. **Commercial use is not permitted under this repository license.**

See [`LICENSE`](LICENSE) for the license notice and the canonical license URL.
