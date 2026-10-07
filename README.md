# ClownGrep

ClownGrep is a local Python utility for matching domains associated with customers and third parties inside text, data-leak-style and modern Excel datasets.

It reads a local `clienti.csv`, recursively scans supported files under `file/`, and writes matched content into separate customer/third-party output folders plus a per-input report.

> **Privacy note:** the `clienti.csv` included in this repository contains only fictional example data. Replace it locally with your own mapping only in a private working copy. Raw datasets placed under `file/`, generated outputs and local virtual environments are excluded from Git by default.

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

You normally do **not** need to install `openpyxl` manually.

If `.xlsx` or `.xlsm` files are detected and the dependency is missing, ClownGrep creates a private `.clowngrep_venv`, installs the required Excel dependency there and restarts itself.

This avoids modifying Homebrew/system Python installations protected by PEP 668.

Manual dependency installation remains possible:

```bash
python3 -m pip install -r requirements.txt
```

## Quick start

The repository already includes a fictional `clienti.csv` that can be used to test ClownGrep.

To use ClownGrep with your own mappings, edit or replace `clienti.csv` **only in a private working copy**.

Place the files to scan under:

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

Each line is treated as an independent record.

If a target domain is detected, only the matching line is saved.

Use this mode for:

- CSV files
- SQL dumps
- JSON / JSONL
- line-oriented logs
- credential lists
- TXT files where one record equals one line

Example:

```text
user=mario | host=portal.example.com | status=active
user=luca  | host=unrelated.example  | status=active
```

Only the first line is saved if `portal.example.com` matches a configured customer or third party.

### 2 — BLOCCHI

Available only for `.txt` and `.log`.

Multiple consecutive lines form one record, while blank lines separate different records.

If any line inside a block contains a target domain, the **entire block** is saved.

Example:

```text
URL: portal.example.com
User: mario
Password: example-password
Browser: Chrome

URL: unrelated.example
User: luca
Password: another-example
```

If `portal.example.com` matches a configured domain, ClownGrep saves:

```text
URL: portal.example.com
User: mario
Password: example-password
Browser: Chrome
```

**Important:** BLOCCHI depends on blank-line separators. Without them, an entire file may be interpreted as one block.

### 3 — SCEGLI PER FILE

Available only for `.txt` and `.log`.

Use this mode when files with the same extension have different internal structures.

For example:

```text
credentials.txt      -> RIGHE
browser_export.txt   -> BLOCCHI
notes.txt            -> skip
```

ClownGrep asks which mode to use for each individual TXT or LOG file.

A single file can also be skipped without excluding all files with the same extension.

### Help

At any mode prompt, type:

```text
h
```

to display examples and a quick explanation.

Practical rule:

```text
one record per line                     -> 1
one record across multiple lines        -> 2
same extension, mixed file structures   -> 3
```

See [`GUIDA_USO.md`](GUIDA_USO.md) for more detailed examples.

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

Customer matches are written under:

```text
outputs/clienti/
```

Third-party matches are written under:

```text
outputs/terze_parti/
```

Per-input reports contain:

- source file
- matched domain
- associated customer or third party
- matched content

Treat generated outputs as potentially sensitive.

## Safe example dataset

A complete synthetic test pack is included under `examples/`:

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

All organizations, domains and records contained in the example dataset are fictional.

The examples use the reserved `.example` namespace.

To run the example dataset:

```bash
cp examples/clienti.csv clienti.csv
cp -R examples/file/. file/
python3 clowngrep.py
```

Suggested processing modes are documented in:

```text
examples/TEST_INSTRUCTIONS.md
```

Expected results are documented in:

```text
examples/EXPECTED_MATCHES.md
```

## Privacy and repository safety

The `clienti.csv` distributed with this repository contains **fictional data only** and is intentionally versioned as part of the public example configuration.

If you replace it with real customer information in your local working copy, do not commit or push that modified file.

The repository configuration excludes operational data such as:

- files placed under the root `file/` working directory
- generated `outputs/`
- `.clowngrep_venv/`
- Python caches
- common editor and operating-system metadata

Synthetic files under `examples/` are intentionally versioned and are safe for public testing.

Before every commit, always verify:

```bash
git status
```

If you use ClownGrep operationally with real customer mappings, the safest setup is to keep a separate private working copy:

```text
~/GitHub/clowngrep/
    clienti.csv        -> fictional public example

~/Tools/clowngrep/
    clienti.csv        -> private operational data
```

Do not publish:

- real customer mappings
- raw leak material
- credentials
- personally identifiable information
- API keys
- access tokens
- internal infrastructure information
- confidential datasets

This applies to commits, issues, pull requests and screenshots.

## Security and scope

ClownGrep is a local triage utility.

It does not:

- download datasets
- authenticate to external services
- exploit systems
- scan remote infrastructure
- perform network attacks

Use it only on data you are authorized to access and process.

See [`SECURITY.md`](SECURITY.md) for responsible-reporting guidance.

## License

ClownGrep is distributed under the **PolyForm Noncommercial License 1.0.0** (`PolyForm-Noncommercial-1.0.0`).

Noncommercial use, modification and distribution are permitted subject to the license terms.

**Commercial use is not permitted under this repository license.**

See [`LICENSE`](LICENSE) for the complete license terms.
