# ClownGrep synthetic test pack

All names, domains and records in this directory are synthetic. The `.example` namespace is reserved for documentation and testing.

## How to use

From the repository root:

```bash
cp examples/clienti.csv clienti.csv
rm -rf file
mkdir file
cp -R examples/file/. file/
python3 clowngrep.py
```

Suggested choices:

| Extension | Choice | Purpose |
|---|---:|---|
| `.csv` | `1` | line-oriented test |
| `.json` | `1` | line-oriented test |
| `.log` | `2` | block-mode test |
| `.sql` | `1` | line-oriented test |
| `.txt` | `3` | demonstrate per-file selection |
| `.xlsm` | `1` | Excel test |
| `.xlsx` | `1` | Excel test |

When `.txt` is set to `3`:

- `01_dataleak.txt` -> choose `1` (RIGHE)
- `nested/06_nested.txt` -> choose `1` for the intended line-oriented test; choosing `2` is useful only to demonstrate the single-block warning

## Customer mapping under test

- Acme Industries -> `acme-industries.example`
  - third party: CloudBox SaaS -> `cloudbox.example`
- Northstar Bank -> `northstar-bank.example`
  - third party: PayBridge -> `paybridge.example`
- BlueRail Logistics -> `bluerail.example`
  - third party: TrackOps -> `trackops.example`

## Behaviours being tested

- exact customer-domain matching
- third-party matching
- subdomain -> configured parent-domain matching
- case-insensitive matching
- non-matching lookalike domain (`evilacme-industries.example`)
- multiple owners in one blank-line-delimited block
- recursive input discovery (`file/nested/06_nested.txt`)
- CSV, JSON, SQL, TXT, LOG, XLSX and XLSM handling

## Important

The input contains a deliberate lookalike:

```text
evilacme-industries.example
```

It must **not** be attributed to `acme-industries.example`.

The LOG file should be processed in BLOCCHI mode. `BLK-004` contains no configured domain and should not appear in extracted results.
