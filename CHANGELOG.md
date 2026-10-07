# Changelog

## 1.0.0

Initial public release of ClownGrep. This release incorporates all functionality, reliability improvements and UX refinements completed before the first GitHub publication.

### Core scanning

- Recursive scanning of supported files under `file/`.
- Customer and third-party domain mapping from `clienti.csv`.
- Exact-domain and subdomain-to-owner matching.
- Supported formats: `.txt`, `.log`, `.csv`, `.sql`, `.json`, `.xlsx`, `.xlsm`.
- Legacy `.xls` is intentionally unsupported because `openpyxl` does not read the old binary format.
- `ripgrep` (`rg`) is preferred for fast searching, with `grep` fallback when available.

### Processing modes

- `RIGHE / dataleak`: treats each line as an independent record and saves only matching lines.
- `BLOCCHI`: for `.txt` and `.log`, treats blank-line-delimited multi-line records as units and saves the full matching block.
- `SCEGLI PER FILE`: for mixed `.txt`/`.log` datasets, allows RIGHE/BLOCCHI selection separately for each file.
- Per-file skip option without excluding the whole extension.
- Interactive `h` help with examples.
- Explicit warning that BLOCCHI requires blank-line separators; otherwise a whole file may be interpreted as one block.

### Excel support

- Read-only processing for `.xlsx` and `.xlsm` through `openpyxl`.
- Automatic local dependency bootstrap: if Excel files are present and `openpyxl` is missing, ClownGrep creates `.clowngrep_venv`, installs the required dependency there and restarts itself.
- Compatible with Homebrew/PEP 668 Python setups without modifying the system interpreter or using `--break-system-packages`.
- Excel progress output includes workbook opening, text conversion and domain-search stages.
- Temporary Excel text files are removed after processing.

### Reliability and safety

- Private temporary domain-pattern files are deleted after execution instead of being left in the project directory.
- Configured domains are validated before processing.
- Conflicting ownership mappings for the same domain are rejected instead of being silently overwritten.
- Safer path-component sanitization, including traversal-like components and Windows reserved names.
- Symbolic links inside the input tree are skipped.
- A symlinked `outputs/` directory is refused before destructive cleanup.
- Runtime customer data, raw input files, generated output, local virtual environments and caches are excluded through `.gitignore`.

### Performance

- `rg`/`grep` results are streamed instead of buffering the full result set in memory.
- TXT/LOG block processing is streamed rather than loading complete files into RAM.
- Report sections are streamed through temporary files rather than retaining all report entries in memory.
- Simultaneously open customer output files are bounded with an LRU handle pool.

### Output and UX

- Separate output trees for direct customers and third parties.
- Per-input reports containing source file, matched domain and matched text.
- Cleaner Excel progress output and end-of-sheet reporting.
- Detailed `GUIDA_USO.md` explaining RIGHE, BLOCCHI and SCEGLI PER FILE.
- Synthetic example dataset included under `examples/` for safe testing.

### Project quality

- Unit tests included under `tests/`.
- GitHub Actions workflow for automated compile/test checks across multiple Python versions.
- `SECURITY.md` with responsible-reporting and sensitive-data guidance.
- Licensed under PolyForm Noncommercial License 1.0.0: noncommercial use, modification and distribution are permitted under its terms; commercial use is not permitted under this repository license.
