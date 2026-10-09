# TypingMaster 12.0.1.981 — Reverse-Engineering Analysis

A documented reverse-engineering study of **TypingMaster 12.0.1.981**, a
32-bit Windows / Delphi VCL touch-typing tutor.

**This repository contains the analysis only.** The original program files,
Wine runtime, extracted PE resources, the live-memory dump and the
decompilation are **not** here — they are third-party proprietary software and
cannot be redistributed. See [`NOTICE.md`](NOTICE.md).

**Community help is welcome** — see [`CONTRIBUTING.md`](CONTRIBUTING.md) and
[`docs/COMMUNITY.md`](docs/COMMUNITY.md).

---

## Key findings

### The executable is packed

The distributed `TypingMaster.exe` (4 MB, PE32) carries only a **97 KB loader
stub** on disk:

- junk-jump obfuscation, just 5 DLL imports (including `crypt32`),
- 2.8 MB encrypted overlay past the section table — **entropy 8.000**, no
  UPX / ASPack / known packer marker ⇒ a **custom protector**,
- static disassembly of the file is useless: only **3 functions** visible.

### It was beaten dynamically

The running application has a decrypted image mapped as anonymous `rwxp` at
`00401000-009b7000`. A live memory dump (`gdb`) captured it; entropy dropped
**8.000 → 6.5** = real code and data. That dump was decompiled with Ghidra
12.1.4 → **13,137 functions**.

> **Address mapping used throughout:** addresses in the decompile are
> *file-relative*. Live address = file offset **+ 0x401000**.

### Application shape

| Metric | Value |
|---|---|
| Framework | Delphi VCL (35 DFM screens, `T*Stg` / `T*Dlg`) |
| Classes recovered from RTTI | **143** |
| Event handlers recovered | **251** |
| Decompiled functions | **13,137** (13,126 OK) |
| Decompile **address coverage** | ~**33 %** of the code image |
| PE resources | 161 (35 screens, 46 bitmaps, 16 cursors, 1 style) |

The low coverage number is the headline result: most of the application lives
in decompiler gaps. The analysis document maps those gaps and recovers names
from Delphi RTTI.

### Licensing — documented, not bypassed

The licence check was **mapped structurally** (handlers, storage fields,
endpoints) and deliberately **not** broken. No crypto/hash strings exist in the
image, so the verification algorithm was not isolated. No bypass is provided
here and none will be accepted as a contribution.

---

## What is in this repository

| Path | Contents |
|---|---|
| `analysis/ANALYSIS-DEEP.md` | Deep technical report (~586 lines) |
| `analysis/classes.txt` | 143 classes recovered from Delphi RTTI |
| `analysis/handlers.txt` | 251 event handlers recovered from RTTI |
| `analysis/form_*.txt` | Per-screen extracts (documented as empty — see below) |
| `DOCUMENTATION.md` | Full narrative write-up |
| `MANIFEST.txt` | MD5 hashes of the 4,240 original files (hashes only) |
| `docs/COMMUNITY.md` | Community channels and most-needed tasks |
| `ROADMAP.md` | Planned work and out-of-scope |
| `CHANGELOG.md` | Version history |

### Known limitations

- `analysis/form_*.txt` extracts are empty: the DFM blobs are stored
  **compressed** in the PE resource, so the extracted text is blank.
- The decompile resolves only ~33 % of the code image by address; the largest
  gap is ~146 KB (`FUN_0049196c` → `FUN_004b5584`).
- The exact WPM/accuracy arithmetic is not pinned down (candidate:
  `FUN_003ba4ac`, float constants 25 / 50 / 100 / 125).

---

## Method

1. Identify: `file`, `objdump -p`, `strings`, a Python PE parser → PE32, 5 DLL
   imports, encrypted overlay (entropy 8.000).
2. Triage: rizin — format, entry point, strings, stub decompilation.
3. Live capture: run under Wine, locate the decrypted `rwxp` mapping,
   `gdb`-dump three regions with the process still running.
4. Decompile: Ghidra 12.1.4 headless (custom `DumpAll.java`).
5. Map: mine RTTI for class and method names; cross-check against the manual.
6. Verify: `md5sum` against a manifest of all original files.

## Contributing

Community contributions are wanted — analysis corrections, cross-distro
testing, documentation, translations and tooling. Start here:

- [`CONTRIBUTING.md`](CONTRIBUTING.md) — every way to help
- [`docs/COMMUNITY.md`](docs/COMMUNITY.md) — most-needed tasks
- Issues labelled **`good first issue`** and **`help wanted`**

## Legal

The analysis and documentation here are the author's original work
(`LICENSE`). TypingMaster itself is commercial, proprietary software by its
respective rights holder and is **not** included or redistributed in this
repository. See [`NOTICE.md`](NOTICE.md).
