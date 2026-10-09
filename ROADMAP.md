# Roadmap

Living document. Items are indicative, not commitments.

## Done — v1.0.0 (2026-10-09)

- [x] Capture the complete program tree
- [x] Identify the framework and the packer
- [x] Defeat the packer dynamically (live memory dump)
- [x] Decompile the unpacked image (Ghidra)
- [x] Map classes and event handlers from Delphi RTTI
- [x] Extract and catalogue PE resources
- [x] Write full documentation
- [x] Publish the study with governance files and CI

## Near term

- [ ] Fill in the largest decompiler gap
      (`FUN_0049196c` → `FUN_004b5584`, ~146 KB) so the ~67 % of the code
      image currently missing from the listing is accounted for.
- [ ] Fully characterise the `.exi` / `.exm` / `.cnt` / `.hdr` / `.wzd`
      lesson formats (field-level).
- [ ] Confirm the exact WPM / accuracy arithmetic (currently `FUN_003ba4ac`
      with the 25/50/100/125 float constants).
- [ ] Document each of the 35 screens with its controls and handler wiring.

## Later

- [ ] Recover readable names for the most-referenced `FUN_*` functions.
- [ ] A script that checks the analysis for internal consistency and dead links.
- [ ] Notes on the satellite (multi-seat) IPC protocol.

## Explicitly out of scope

- Breaking, patching or bypassing the licence check (structurally documented
  only).
- Redistribution of the proprietary product. This repository contains the
  analysis only; the software itself is not distributed here.
