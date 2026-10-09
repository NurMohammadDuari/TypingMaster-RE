# Changelog

All notable changes to this repository are recorded here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-10-09

First public release of the reverse-engineering **analysis**.

### Added

- **Deep report** `analysis/ANALYSIS-DEEP.md` (~586 lines): custom-packer
  analysis, live-memory capture method, decompile coverage (13,137 functions,
  ~33 % resolved by address), RTTI-based name recovery, application data
  formats, PE resource catalogue, and the launcher / launch chain.
- **RTTI maps**: `analysis/classes.txt` — 143 classes; `analysis/handlers.txt`
  — 251 event handlers.
- **Per-screen extracts** `analysis/form_*.txt` (empty by nature — the DFM
  blobs are stored compressed in the PE resource).
- **`DOCUMENTATION.md`** — narrative write-up with a table of contents and a
  verified inventory appendix.
- **`MANIFEST.txt`** — MD5 **hashes only** of the 4,240 studied files, kept as
  a provenance and integrity record.
- **Governance and community files**: `LICENSE`, `NOTICE.md`,
  `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, `SUPPORT.md`,
  `AUTHORS.md`, `MAINTAINERS.md`, `ROADMAP.md`, `CITATION.cff`.
- **Docs index and community guide**: `docs/README.md`, `docs/COMMUNITY.md`.
- **Repository hygiene**: `.gitattributes` (line-ending protection so manifest
  hashes stay valid), `.gitignore`, `.editorconfig`, `.gitmessage`.
- **`.github/`**: issue forms, pull-request template, `CODEOWNERS`, labeler,
  docs-lint and stale workflows.

### Notes / limitations

- The studied `TypingMaster.exe` is protected by a custom packer (97 KB loader
  stub; encrypted overlay, entropy 8.000). Static analysis of the file on disk
  is not useful; all results come from the live memory dump.
- Addresses in the decompilation are file-relative: live-memory address =
  file offset + `0x401000`.
- The decompile resolves only about 33 % of the code image by address;
  `ANALYSIS-DEEP.md` documents the gaps and the RTTI-based name recovery.
- DFM form resources could not be read from the PE (stored compressed), so the
  `analysis/form_*.txt` extracts are empty.
- The licensing check was **mapped, not broken**. No bypass was implemented and
  none will be accepted as a contribution.

### Legal

- This repository contains the **analysis only**. The proprietary program, its
  assets, the Wine runtime and the decompiled listing are **not** included and
  are not redistributed. See `LICENSE` and `NOTICE.md`.
