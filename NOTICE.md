# NOTICE — Attribution and scope

This repository is a **reverse-engineering study**. It documents how a
third-party application was examined. It does **not** bundle that application.

## 1. The analyzed target (not included)

**TypingMaster 12.0.1.981** is a commercial, 32-bit Windows application
(Delphi VCL) by its respective rights holder.

- It is **not** present in this repository.
- No rights to it are granted here.
- Only its observed structure, class/method names recovered from its own
  RTTI, file-format observations, and hashes of its files appear here.

The following were used during the study but are deliberately **excluded**
from this public repository, because they are third-party work that cannot be
redistributed:

| Excluded material | Why |
|---|---|
| Program tree (program, lessons, sounds, images, manuals) | Proprietary third-party software |
| Extracted PE resources (screens, cursors, bitmaps, style) | Derived from the above |
| Live-memory dump of the decrypted image | Derived from the above |
| Ghidra decompilation | Derived from the above |
| Wine runtime and platform libraries | Third-party licensed binaries |
| Original launcher scripts from the software package | Third-party work |

## 2. What this repository does contain

- Analysis, notes, class/handler name lists and reports authored by the
  maintainer.
- `MANIFEST.txt` — **hashes only** of the studied files, as a provenance and
  integrity record. Hashes are not the files and reveal no content.

## 3. Tools referenced

The study used `file`, `objdump`, `strings`, Python, rizin, Ghidra, `wrestool`
and `gdb`. These are independent projects and are neither bundled nor
affiliated with this repository.

## 4. Author's own work

Everything in this repository is the maintainer's original work unless stated
otherwise, and is covered by `LICENSE` (All Rights Reserved).

## 5. Requests

Rights holders who believe something here is inappropriate can open an issue
or contact [@NurMohammadDuari](https://github.com/NurMohammadDuari); it will
be addressed promptly.
