# TypingMaster 12 — Complete Documentation (First to Last)

## Contents

1. [What this is](#1-what-this-is)
2. [How it was examined (reverse engineering)](#2-how-it-was-examined-reverse-engineering)
3. [What is in this folder](#3-what-is-in-this-folder)
4. [How to run it](#4-how-to-run-it)
5. [How it works at launch (chain of events)](#5-how-it-works-at-launch-chain-of-events)
6. [License key (how it works here)](#6-license-key-how-it-works-here)
7. [What this folder is and is not](#7-what-this-folder-is-and-is-not)
8. [How it was done (methods, step by step)](#8-how-it-was-done-methods-step-by-step)
9. [Maintenance](#9-maintenance)
10. [Appendix: verified inventory](#10-appendix-verified-inventory)

## 1. What this is

TypingMaster 12 is a commercial touch-typing tutor made by Cloud Kayak
Labs. It teaches typing through courses, lessons, drills, tests, games,
statistics, and a background typing monitor. It is a 32-bit Windows
program (written in Delphi). New copies come with a 7-day free trial;
continued use of all features requires a paid license key.

On this Linux PC it runs through Wine (a Windows-compatibility layer),
delivered as a Flatpak package (`com.typingmaster.TypingMaster`).

## 2. How it was examined (reverse engineering)

1. **Installed** the Flatpak (`TypingMaster-12.0.1.981-x86_64.flatpak`).
   Found the program tree: a 4 MB `TypingMaster.exe` plus 131 support
   files (lessons, sounds, pictures, keyboard layouts).
2. **Identified the framework**: Delphi VCL — 35 screen layouts (DFM
   forms), style resources, `T*Stg/T*Dlg` screen classes.
3. **Found the packer**: the exe on disk shows only a 97 KB loader stub
   (junk-jump obfuscation, 5 system imports including `crypt32`).
   2.8 MB of encrypted overlay (entropy 8.000, no known packer markers
   = custom protector). Static analysis of the file is nearly useless
   (only 3 functions visible).
4. **Beat the packer dynamically**: ran the app, found the decrypted
   5 MB image in anonymous memory (`00401000-009b7000`), dumped it with
   gdb (`unpacked/`). Entropy dropped to 6.5 = real code and data.
5. **Decompiled** the dumped image with Ghidra 12.1.4:
   `decompiled_typingmaster.c` — 13,137 functions, 13,126 OK.
6. **Mapped the UI**: 35 screens, 251 button/event actions, 143 classes
   (`analysis/`). Verified flows against the official user manual and
   13 reference screenshots (login, course menu, drills, review, test,
   results, games, statistics, settings).
7. Tools used: rizin (triage/decompile), Ghidra (batch decompile),
   IDA Free 9.4 (installed + licensed, but its Linux build cannot run
   scripts, so it served manual browsing only), wrestool (resources),
   gdb (live memory dump).

## 3. What is in this folder

| Path | Contents |
|---|---|
| `analysis/` | Class map, handler map, deep technical report, per-screen extracts |
| `DOCUMENTATION.md` | This file |
| `README.md` | Short map of the repository |
| `MANIFEST.txt` | MD5 hashes of the 4,240 studied files (hashes only) |
| `docs/` | Community guide and documentation index |
| `ROADMAP.md`, `CHANGELOG.md` | Plans and version history |
| `LICENSE`, `NOTICE.md` | Terms and attribution |
| `CONTRIBUTING.md`, `SECURITY.md`, `SUPPORT.md`, `CODE_OF_CONDUCT.md` | How to take part |

> **Not included:** the program tree, Wine runtime, Wine prefix, extracted PE
> resources, the live-memory dump and the decompilation. Those are third-party
> proprietary material and cannot be redistributed. See `NOTICE.md`.

## 4. How it was run

The study reproduced the program under Wine. The runnable tree is **not**
distributed here (see §7). The original run used a standalone launcher that:

1. set `PATH` / `LD_LIBRARY_PATH` to the bundled Wine engine and libraries,
2. pointed `WINEPREFIX` at a local prefix,
3. copied the program tree into the prefix, applied DPI scaling, and launched
   `TypingMaster.exe`.

The first run builds the Wine prefix (about a minute); internet is not
required.

## 5. How it works at launch (chain of events)

1. Launcher sets `PATH`/`LD_LIBRARY_PATH` to the folder's Wine + libs.
2. Points `WINEPREFIX` at the folder's `prefix/`.
3. Original launcher copies `app/` files in, applies DPI scaling.
4. `TypingMaster.exe` starts, decrypts itself in memory.
5. Login screen → lessons, tests, games, statistics all read from
   these files. Progress is saved under `prefix/`.

## 6. License key (how it works here)

7-day free trial, then courses lock. About menu → Enter License Key
(`TProductKeyDlg`): enter a purchased License ID + key → green
confirmation, features unlock. The check itself lives in the encrypted
code; it was mapped, not broken. A valid bought key activates here
exactly like the store app.

## 7. What this repository is and is not

- It IS: a documented reverse-engineering study — findings, maps, reports
  and reproducible methodology.
- It IS NOT: the company's original source code, nor a copy of the product.
  The program, its assets, the Wine runtime and the decompiled listing are
  deliberately **not** distributed here.
- Legal boundary: the studied product is commercial and proprietary. This
  repository publishes only the author's own analysis and observations, and
  does not redistribute the product or bypass its licensing.

## 8. How it was done (methods, step by step)

- **Install & locate**: `flatpak install --user <bundle>`; program
  tree found under the app's `files/typingmaster/`, Wine tools under
  `files/bin/`, launcher at `files/bin/typingmaster` (read it first —
  it documents prefix handling, DPI logic, and env overrides).
- **Identify**: `file` (PE32), `objdump -p` (5 DLL imports),
  `strings` (VCL/style markers, 19k strings), PE section parse in
  Python (found 2.8 MB overlay past section data, entropy 8.000 =
  encrypted; no UPX/aspack markers = custom protector).
- **Triage via rizin MCP**: `triage_binary` (format, entry point,
  strings); `decompile_function` on stub addresses (showed
  junk-jump obfuscation + decrypt loop).
- **Resources**: `wrestool -l/-x` → 35 DFM forms (stored compressed),
  bitmaps, cursors, VCL style, version info.
- **Live capture**: `flatpak run` the app; found decrypted image as
  anonymous `rwxp` mapping `00401000-009b7000`; `gdb -p <pid> -batch
  -ex "dump memory ..."` for 3 regions (process kept running).
- **Decompile**: Ghidra 12.1.4 headless (`analyzeHeadless`) with a
  custom `DumpAll.java` script (decompiles every function to one
  `.c` file); raw-binary import for the memory dump.
- **UI mapping**: strings/RTTI mining from the dump (class names,
  event handlers like `OKBtnClick`), cross-checked with the official
  user manual and 13 reference screenshots (login, menu, drills,
  review, test, results, games, statistics, settings).
- **Standalone bundle**: copied Flatpak's Wine (`app-files/`) and
  Platform runtime (`platform-files/`) into `runtime/`; wrote
  `run-standalone.sh` (`PATH`/`LD_LIBRARY_PATH`/`WINEPREFIX` +
  original launcher via env overrides); verified by screenshot and
  exe hash match.
- **Verify everything**: screenshot each stage; `md5sum` comparisons;
  unit-tested rebuilt components; `MANIFEST.txt` over all files.

## 9. Maintenance

- This repository holds the analysis. The studied tree itself lives in a
  private companion repository; verify it there with
  `md5sum -c MANIFEST.txt`.
- `MANIFEST.txt` here is the provenance record of the studied files — hashes
  only, no content.
- Keep the analysis documents in sync if a claim is corrected.

## 10. Appendix: verified inventory

Counts measured directly against the studied tree (`find`, `du`, `wrestool`,
RTTI mining), not estimated. They describe the **private** companion tree, not
the contents of this repository.

| Item | Verified value |
|---|---|
| Total files in tree | 33,150 |
| Total size | 1.9 GB |
| Symlinks (Wine runtime) | 2,670 |
| Largest single file | 85.5 MB (`prefix/.../93d2.msi`) — under GitHub's 100 MB cap |
| `app/` files | 4,076 (23 MB) |
| `app/lessons/` files | 3,648 |
| `app/texts/` exercise texts | 230 (`.exi` + `.exm` pairs) |
| `app/keyboards/` layouts | 60+ (`.kbd`) |
| `app/manual/` pages | 12 (HTML + CSS) |
| `runtime/` files | 20,006 (1.1 GB) |
| `prefix/` files | 8,858 (768 MB) |
| PE resources extracted | 161 |
| DFM screens | 35 |
| Bitmaps / cursors / icons | 46 / 16 / 1 |
| VCL styles | 1 (`VCLSTYLE_WINDOWSDARK`) |
| Classes recovered from RTTI | 143 (`analysis/classes.txt`) |
| Event handlers recovered | 251 (`analysis/handlers.txt`) |
| Decompiled functions | 13,137 (13,126 decompiled OK) |
| Manifest entries | 4,240 |

### PE resource types present

`RT_BITMAP` (46), `RT_CURSOR` (16), `RT_ICON`/`RT_GROUP_ICON` (main icon),
`RT_STRING` (8 tables), `RT_DIALOG`/DFM forms (35), `RT_VERSION`, `RT_RCDATA`
(VCL style + platform targets), plus the standard `BBOK`/`BBYES`/`BBABORT` …
button-image resources and `MSG_ERROR` / `MSG_INFO` / `MSG_WARNING` glyphs.

### Known quirks

- `analysis/form_*.txt` extracts are empty (1 byte) for most forms: the DFM
  blobs are stored **compressed** in the PE resource, so the extracted text is
  blank. The real screen content lives in the decrypted dump instead.
- `native-re/functions.txt` lists only the packer stub (120 bytes) — a
  consequence of the protector, not a failed extraction. Use
  `native-re/decompiled_typingmaster.c` for the real code.
- Addresses in the decompile are file-relative; add `0x401000` to get
  live-memory addresses.
