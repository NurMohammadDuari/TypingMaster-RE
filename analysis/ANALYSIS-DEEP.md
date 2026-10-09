# TypingMaster 12.0.1.981 — Deep Technical Analysis

Reverse-engineering report over the tree `TypingMaster-Source/`.
Everything below is derived from the artifacts in this repository only; no network
access, no execution of the product, and no modification of any input file.

**Convention used throughout**

| Context | Address space |
|---|---|
| Ghidra decompile names (`FUN_XXXXXXXX`) | **file-relative**: `unpacked/code_401000.bin` offset |
| Live process memory / RTTI pointers stored in the binary | **live** = file offset + `0x401000` |
| Strings quoted as "file 0x…" | offset inside `unpacked/code_401000.bin` |
| Strings quoted as "live 0x…" | file offset + `0x401000` |

Work was done with grep/ripgrep plus throw-away Python scripts written to
`/tmp/opencode/` (outside the protected tree). Only this file was created.

---

## 1. Scope of the decompiled C

`native-re/decompiled_typingmaster.c`

* **12,230,838 bytes**, **502,553 lines**, **13,137 function headers**.
* No real symbol names survive: every header is `FUN_XXXXXXXX`, `caseD_XXXXXXXX`
  or `thunk_XXXXXXXX`. The only human-readable identifiers inside function bodies
  come from RTTI strings and string literals that the decompiler echoed.
* Function headers are all at column 0 and all consist of a `void`-returning
  no-parameter prototype, i.e. Ghidra's type recovery produced no signatures —
  parameters and return types were not recovered, so all argument passing appears
  as `*(undefined4 *)param_1` style pointer chasing.

### 1.1 Address coverage (quantitative)

Sum of function body sizes = `0x1E244D` (1,975,373 B) against a code image of
`0x5B6810` (5,990,416 B) → **only 33.0 % of the code address range is covered by
decompiled functions.** The remaining 67 % is either data, padding, or — most
importantly — large stretches Ghidra never turned into functions.

Largest single gap: **`0x23BDA` (146,394 B)** between the end of `FUN_0049196c`
and the start of `FUN_004b5584`. Many application-level routines live inside such
gaps (see §4 and §5 — every product-key handler falls in one).

Largest functions by body size:

| Function | Bytes | Note |
|---|---|---|
| `FUN_003ac22c` | 18,629 | satellite / telemetry / uninstall-URL builder (§4.6) |
| `FUN_0043eaf0` | 13,151 | — |
| `caseD_0` @ `0x204b0` | 9,284 | contains SysUtils exception RTTI → RTL, not app logic |
| `FUN_00536ce8` | 8,489 | — |
| `FUN_003f9968` | 7,117 | — |

Most-called functions are all RTL plumbing, not product logic:
`FUN_0000a08c` (1,845 refs), `FUN_000088a8` (1,274), `FUN_00008900` (1,114).

### 1.2 Data labels

* 1,114 distinct `DAT_*` labels overall, but only **95 inside the code region** —
  the rest are in the data/RTTI areas.
* 16 `s_*` string labels, e.g. `s_TCRCStreamH_0040e5e4`,
  `s_ReadRecString25N_00252565`, `s_UGamesView_0053d5b6`. These are the only
  places where Ghidra attached a *source-level* identifier (`TCRCStream`,
  `UGamesView`) — i.e. residual Delphi unit/class names embedded as string
  constants.

### 1.3 String cross-reference map

A pointer scan for little-endian 32-bit values pointing into the code image
produced `/tmp/opencode/xref_map.txt`: **2,130 distinct functions reference at
least one string literal.** That map is the backbone of the subsystem inventory
in §4, because function names are absent but string payloads are not.

---

## 2. Architecture recovered from the decompile

The binary is a **Borland/Embarcadero Delphi Win32 application** built with the
VCL. Evidence:

* `caseD_*` dispatch tables, `DAT_*` RTTI blobs and `undefined1 [N]` filler are
  characteristic of Ghidra's output over Delphi VMT/RTTI sections.
* String literals: `SysUtils` exception machinery, `System.Win.Registry`,
  `GIFImg`, `OleServer`, `System.JSON`, `HTMLSubs`/`HTMLUn2`/`HtmlView`/`vwPrint`.
* Published-method RTTI (`TPublishedMethod`-style records) with
  `(Self)` parameter markers — see §3.
* `.dfm` form resources (§7), `object …: TComponent` naming in `analysis/classes.txt`.

Layering observed (top → bottom):

1. **Forms / controllers** — 143 classes (`analysis/classes.txt`), 251 event
   handlers (`analysis/handlers.txt`).
2. **Domain services** — results store, course/lesson selection, bigram analysis,
   keyboard layout importers, graphics/graph builder.
3. **Infrastructure** — embedded HTML viewer engine, MSHTML bridge, JSON, registry,
   GIF decoder, IPC (satellite process), HTTP (wininet-style URL strings).
4. **RTL** — the ~FUN_00008xxx/FUN_0000axxx cluster (highest call counts).

### 2.1 Subsystem map (string-xref evidence)

| Subsystem | Anchor functions (file-relative) | Key strings |
|---|---|---|
| Course selection / contents | `FUN_0053bc74` | `ChooseCourse`, `CourseContents` |
| Texts path building | `FUN_001741c8`, `FUN_00175358`, `FUN_00346718`, `FUN_003940e4` | `Texts/` |
| Language switching | `FUN_002578d8` | `TMLangs.ini` |
| Keyboard layout load | `FUN_00344840`, `FUN_00345e04` | `TMKeyb.ini` |
| Legacy Word integration | `FUN_0033a04c` | `TypingMaster 98/99`, `WINWORD.EXE`, `Microsoft Word` |
| Satellite / second process | `FUN_003ac22c`, `FUN_0033eca8` | `TM11.TypingMasterSatellite.Running`, `TMSatellite1999.DISABLE/ENABLE`, `TM2001.BRINGFRONT`, `TBookFormEx`, `https://www.typingmagic.com/`, `KBOOST.EXE` |
| Bigram analytics | `FUN_0052068c`, `FUN_00521170`, `FUN_00523cd4` | `TBigramAnalyzer`, `"bigrams": [`, `Bigrams: Difficult/Slowest` |
| KLC keyboard import | `FUN_00546938` … `FUN_0054cb74` | `KLCImporter`, `TKLCKeyMapping`, `TDeadKeyMapping` |
| KMN keyboard import | `FUN_00193270`, `FUN_004f1908`, `FUN_00552298` … `FUN_0056364c` | `KMNImporter` |
| HTML rendering | `FUN_00008728` … `FUN_0050e0e8` | `HTMLSubs`, `HTMLUn2`, `HtmlView`, `vwPrint`, `IHTMLDocument2` |
| JSON encode/decode | `FUN_0038bbce` … `FUN_0038efcc` | System.JSON |
| Registry / images | — | `System.Win.Registry`, `GIFImg`, `OleServer` |
| Product web presence | `FUN_0057aff0` | `https://www.typingmaster.com/` |

---

## 3. Symbol recovery: Delphi published-method RTTI

Because Ghidra produced no names, method names were recovered from the
**published-method RTTI** that Delphi emits next to each method pointer. Layout
found by strict scan:

```
[Len : Word]              // record length, typically N+7 (larger when params exist)
[Code : Pointer]          // LIVE address of the method (file offset + 0x401000)
[NameLen : Byte]
[Name : Char * NameLen]   // e.g. "OKBtnClick", "FormShow", "CheckNetWorkLicenses(Self)"
```

Result: **150 strictly-validated entries, 103 distinct method names**
(union with looser matching: 155 names).

Placement — the single most useful structural finding:

* **39 entries fall inside decompiled functions** (so name ↔ body ↔ strings can be
  correlated directly),
* **111 entries fall inside Ghidra gaps**, i.e. the very routines the decompiler
  missed. This explains why licensing, form handlers and settings code are
  "missing" from `decompiled_typingmaster.c`.

Helper tables: `/tmp/opencode/methods2.txt`, `/tmp/opencode/methods4.txt`,
`/tmp/opencode/funs.txt` (13,137 `addr name size` lines).

---

## 4. Licensing & registration — structural documentation only

> This section documents **where** and **how** license data is represented and
> referenced. It deliberately does **not** attempt, describe or enable any bypass,
> key generation, patching or circumvention. The verification algorithm itself
> was **not isolated** (see §11).

### 4.1 The product-key dialog class

`TProductKeyDlg` (in `analysis/classes.txt`), components:

`ProductKey`, `LicenseID`, `CurrentLic`, `LicenseIDEdit`, `ProductKeyEdit`,
`LicensesList`, `Instruction`, `EnterNew`, `Label1`, `RegHardwareID`, `lcounter`,
`wcounter`, `thankyou`.

Handler → address (live, as stored in RTTI; file-relative in parentheses):

| Handler | Live | File (`FUN_` space) |
|---|---|---|
| `TProductKeyDlg.OKBtnClick` | `0x7A2F60` | `0x3A1F60` |
| `TProductKeyDlg.FormShow` | `0x7A3F70` | `0x3A2F70` |
| `TProductKeyDlg.FormCreate` | `0x7A4088` | `0x3A3088` |
| `TProductKeyDlg.LicenseIDEditChange` | `0x7A426C` | `0x3A326C` |
| `TProductKeyDlg.LicensesListClick` | `0x7A4280` | `0x3A3280` |
| `TProductKeyDlg.CancelBtnClick` | `0x7A434C` | `0x3A334C` |
| (license entry button) `LicenseBtnClick` | `0x7AA8AC` | `0x3A98AC` |
| `LearnMoreBtnClick` | `0x7E1FA8` | `0x3E0FA8` |

**All of these fall in decompiler gaps.** Concretely: `FUN_003a1100` ends at
`0x3A1267`, the next function starts at `0x3A4A1C` → a **14,261-byte hole** that
contains `OKBtnClick`, `FormShow`, `FormCreate`, `LicenseIDEditChange`,
`LicensesListClick` and `CancelBtnClick`.

### 4.2 License storage / status RTTI field names

Recovered verbatim from RTTI blobs:

`LoadMagFile`, `XXXsetRegistrationStatus`, `xxxAdd`, `XXXList`, `Load`, `TWSL`,
`WSLic`, `AStatus`, `ownerName`, `PKey`, `LicID`, `MagKeyFilename`, `XMode`,
`proTr`, `Conc`, `keys`, `users`, `vStatus`, `vOwnerName`, `thankyou`.

Interpretation (**inference**): a persistent record keyed by owner name +
product key + license ID, with a status word (`AStatus`/`vStatus`), persisted
through a "mag file" (`MagKeyFilename`), supporting multiple stored licenses
(`XXXList`, `LicensesList` UI control) and network/concurrent seats
(`Conc`, `proTr`).

### 4.3 Network licensing settings

`TProgramSettings` RTTI (file `0x34440A`–`0x34453E`, live `0x74540A`+):

* `CheckNetWorkLicenses(Self)`
* `FreeNetWorkLicense(Self, amount)`
* `NetworkLicenseFile`
* `NetworkLockByte`

**Inference**: an optional network-licence mode with a lock byte guarding a
shared licence file, checked at startup and released on exit (`amount` suggests
seat accounting).

### 4.4 Magic value and the registration-status helper

* Literal `6486E1A1` at **file `0x345660`** (live `0x746660`).
* Referenced at **file `0x3448B7`**, inside **`FUN_00344840` (3,319 B)**, which
  also references `SetLocation`, `kmgSubMenuGlyph`, `TMKeyb.ini`.
  → **inference**: a 32-bit tag/marker constant processed by the same routine
  that loads keyboard/menu configuration; not, on this evidence, an
  encryption key.
* `TypingMaster licensed to` at **file `0x346A0D`**; referenced at **file
  `0x34690B`** inside **`FUN_00346888` (284 B)**, which also references
  `CallHelp` → **inference**: the small "licensed to <owner>" banner formatter,
  called when showing product/registration info and help.

### 4.5 Online endpoints (URL strings in the image)

| URL fragment | File offset | Referenced at | Containing function |
|---|---|---|---|
| `typing-tutor/12/validate.php?key=` | `0x3A2D98` | `0x3A22F9` | in the gap after `FUN_003a1100` |
| `typing-tutor/12/update-now.php` | `0x39E1B0` | — | `FUN_0039DE44` (630 B) |
| `&tmv=10&be=7&c=` , `&hid=` , `&sid=` | nearby | — | `FUN_0039DE44` |
| `typing-tutor/12/uninstall.php?b=` , `&l=0&dd=-1` , `&mg=1&ev=1` , `&key=` , `.json` , `.wk10` , `12.01.981` | — | — | `FUN_003ac22c` (18,629 B) |

`FUN_0039de44` is confirmed as method **`Googleshow`** (RTTI code `0x79EE44`
= `0x39DE44` + `0x401000`) — name recovered by the §3 technique.

**Note (no attempt made):** `validate.php?key=` exists as a string; nothing in
this report tests or exercises it.

### 4.6 Trial / expiry strings (data side)

From `app/language.eng` (IDs are `language.eng` entry numbers):

| ID | Text (abridged) |
|---|---|
| 107 | 7-day trial explanation |
| 206 | "this copy has expired" |
| 207 | "Limited trial version" |
| 302 | "Free Trial Active" |
| 211/212/213/215/217 | license messages |
| 435 | upgrade coupon `UPG12V11` |
| 716–719 | license dialog copy |

Runtime state names found in the image: `TimeLeft`, `VTimeLeft`, `LastTimeLeft`.
**Inference**: remaining trial time is persisted and re-read (`LastTimeLeft`),
consistent with a countdown that survives restarts.

### 4.7 `app/TM11LICEN.NET` (256 bytes) — raw layout

```
00000000: 18 "TypingMaster licensed to" nen\AppData\Roaming\TypingMaster11\TMLICEN.NET\00
          ^ u8 length = 0x18 (24)        ^ 46-byte NUL-terminated string
then 46 little-endian u32 values, e.g.
  0x02D9158B, 0x00048000, 0x7685AE43, 0x02D9158B, 0x00008040, 0x7685AE68, ...
  0x004021EC, 0x0019F3A0, 0x02ADA83C, 0x00402378, 0x0019F3DC, 0x0040239C,
  0x0040360C, 0x02AE4004, 0x0040398A, 0x004039E2, 0x00403641
```

Observations:

* The trailing path fragment `...\AppData\Roaming\TypingMaster11\TMLICEN.NET`
  is **inference-grade evidence** that the *live* licence file lives under
  `%APPDATA%\TypingMaster11\TMLICEN.NET`, not next to the executable.
* The leading `nen` before `\AppData` is unexplained — **inference**: a
  truncated/overwritten remainder of a longer path (e.g. `C:\Users\…\`).
* Several u32 values land inside the loaded image (`0x004021EC`, `0x00402378`,
  `0x0040239C`, `0x004023A3`, `0x0040360C`, `0x0040398A`, `0x004039E2`,
  `0x00403641`) while others look like heap addresses (`0x0019F3A0`,
  `0x0019F3DC`, `0x02ADA83C`, `0x02AE4004`) and some look like timestamps
  (`0x7685AE43`, `0x7685AE68`).
  → **inference**: this 256-byte file is a *raw memory snapshot* of internal
  structures (pointers intact), not a human-editable config. Its content is
  therefore not plaintext licence text.

---

## 5. Results, WPM and statistics

### 5.1 `TResultItem2` record (RTTI field order, verbatim)

`UniqueID`, `IDFormat`, `DataFormat`, `SubType`, `IDName`, `Course`, `Lesson`,
`Exercise`, `Event`, `Checked`, `DateTime`, `Duration`, `TypingTime`,
`Completed`, `FullTime`, `CumulativeTypingTime`, `GrossSpeed`, `Accuracy`,
`NetSpeed`, `Rhythm`, `Efficiency`, `DiffIndex`, `GrossHits`, `Errors`,
`ErrorFactor`, `Started`, `Passed`, `TestName`, `Score`.

Supporting types:

* `TResults` methods: `Load`, `Save`, `AddResult`, `GetResultIndex`,
  `GetSortedValues`, `GetValues`, …
* `TEfficiencyInfo` accessors: `GetGrossSpeed` (+`Grade`, `GradeValue`),
  `GetAccuracy` (+`Double`, `Grade`, `GradeValue`), `GetNetSpeed`
  (+`Grade`, `GradeValue`), `GetEfficiency` (+`Grade`, `GradeValue`).

**Inference**: grades/grade-values are separate computed fields, i.e. results are
scored against configurable thresholds (cf. `AccuracyGoalList`,
`GoalSpeedMin`, `ErrorFactor=5` in `app/Policy*.dyn`).

### 5.2 Graph accessors → chart builder

| Accessor | Live/file function | Size |
|---|---|---|
| NetSpeed | `FUN_003ba1e8` | 82 B |
| GrossSpeed | `FUN_003ba2d8` | 79 B |
| Accuracy | `FUN_003ba350` | 79 B |

All three call **`FUN_003ba4ac` (2,087 B)** — the shared chart builder. It pads
the sample array to `0x10` (16) entries and materialises float constants:

`0x42FA0000` = **125.0**, `0x41C80000` = **25.0**, `0x42480000` = **50.0**,
`0x42C80000` = **100.0**.

→ **Inference**: 25/50/100 are goal/scale markers on the speed & accuracy charts
(125 likely a max-scale for WPM/KPM). The exact WPM arithmetic was **not**
isolated: floats appear only as hex constants and the routines containing the
multiplication sit partly in Ghidra gaps (see §11).

### 5.3 `TCourseGraphs`

`DiffKeysGraph`, `NetSpeedGraph`, `GrossSpeedGraph`, `AccuracyGraph`,
`EfficiencyGraph`, `RhythmGraph`, `DiffIndexGraph` — i.e. one chart per metric
in `TResultItem2` above.

Related UI strings: `KPM (keystrokes/min)` (language ID 203), `spnMinWPM`,
`lblMinWPM`, `minkpm`, `GoalSpeedMin`, `AccuracyGoalList`.
→ **Inference**: the product counts **keystrokes per minute** internally and
labels it WPM/KPM interchangeably in the UI.

---

## 6. Application data formats (`app/`, 4,076 files, ~23 MB)

### 6.1 `texts/` — 230 files = 115 `.exi` + 115 `.exm`

* `.exi` = **plain text `key=value` metadata**. Keys observed (with frequencies):
  `language` (123), `version` (104), `author` (101), `copyright` (92),
  `description` (70), `name`/`Name`, `category`, `encode`, `time`, `speed`,
  `mustTypeAll`, `duration`, `MinSpeed`, `MinAccuracy`, `sourceFormat`,
  `nameunicode`, `descriptionunicode`.
  Example (`texts/272336d1.exi`): UTF-8 BOM, `encode=utf8`, `#Generated online`,
  `category=science`, `language=hin`, `time=10`, `mustTypeAll=0`.
* `.exm` = the **plain-text body** of the exercise.
→ **inference**: `.exi`/`.exm` are a paired sidecar format (metadata + text),
with `.exi` optionally carrying Unicode variants for non-Latin scripts.

### 6.2 `lessons/` — 3,648 files

| Ext | Count | Format |
|---|---|---|
| `.txt` | 2,739 | plain lesson text |
| `.htm` | 369 | HTML content |
| `.exc` | 351 | plain-text exercise definition |
| `.cnt` | 50 | `$`-prefixed header + `key=value` course table |
| `.int` | 31 + 10 | plain-text tips/interstitial copy |
| `.hdr` | 17 | fixed **10-byte records** |
| `.wzd` | 17 | length-prefixed words + stats tail |
| other | 64 | — |

* **`.cnt`** example (`lessons/1italian.cnt`):
  `$description=`, `$longdesc=`, `$version=1.0`, `$language=IT`, `$chapters=12`,
  `$copyright=TypingMaster inc 2005, all rights reserved.`, then
  `'Created by TMEditor v2.0 @ 14.12.2004 18:48` (apostrophe = comment), then
  bare `key=value` entries: `newkeysloops=4`, `wordloop=2`, `wordmaxloop=0`,
  `contents name=…`, `keyboard=all`, `default time=45`, `wizardname=1ITALIAN`,
  `simplewizard=1`, `last free=2`, `keyboardbase=QWERTY QWERTZ`.
* **`.hdr`** — 10-byte records: `u16 char`, `u16 0x0016`, `u16 1`, `u16 offset`,
  `u16 length`. `1ITALIAN.hdr` = 10,890 B = **1,089 records**; observed char
  set `a,s,d,f,k,l,i,e,n,r,c,o,t`.
  → **inference**: a per-character index into a word/lesson body (offset+length),
  used to pick words containing newly-introduced keys.
* **`.wzd`** — length-prefixed word strings followed by an approximately 24-byte
  statistics tail. `1ITALIAN.wzd` = 105,350 B.
  → **inference**: wizard-mode word pool plus aggregate counters.

### 6.3 `keyboards/` — 56 `.kbd` files (a small DSL)

Versioned text format, `$version=6`:

```
dynamiclayout;1
MaterialBased;dynamic
HomeRow;dynamic
DefaultSize;32;34
FontSize;18;14;14
vKey;finger;x;y;width;height;style;keycaption;…;$scancode
BackSpaceKey … TabKey … SpaceKey … ControlKey … CapsLockKey
' comment
```

Relative coordinates use `+n` deltas and `-` for defaults.

### 6.4 Root-level configuration

| File | Content |
|---|---|
| `course.tpl` | INI-like: `[w]` plus `Setting6..Setting10`, `datatype6..10`, `description10` |
| `tmlangs.ini` | `[active] Language=ENG` + display-name → language-code map |
| `tmnpath.ini` | `[path] AppData=1` + `[ab]` |
| `Policy.dyn`, `Policy-store.dyn`, `Policy-storeus.dyn` | `key=value`: `secondarycourse=`, `SatelliteWizard=`, `DefaultWizard=`, `ErrorFactor=5`, `KeyboardLayout=*101-Keyboard (US)`; the store variants differ in `linkaddress1..3` |
| `0.Grp` … `4.Grp` | training profiles: `Test.Name/Duration/Message/Enabled/ID`, `Name`, `ContentsFile`, `Date=45980.31640625` (Delphi TDateTime), e.g. `2.Grp` = "Junior Easy Course" → `engjr.cnt` |
| `top100difficultwords.txt` | 84 lines |
| `top1000wordlist.txt` | 970 lines |

### 6.5 Localisation

* `language.{eng,fin,fr,ger,it,nl,pt,pt2,spa,swe,tr,tr2}` — UTF-8 **with BOM**,
  `//` comments, `ID=string` entries. `language.eng` has **1,485 entries**;
  header states *"Typing Master 10 GUI Localization … 6.21 … revision 1.2 …
  12/27/2002"* (i.e. the string table lineage predates this build).
  Last IDs: 1912–1915 (exit course / mistyped-word storage consent).
* `language-store.i` (19 entries, incl. **717 = "Enter License Key"**),
  `language-storeus.i`, `language-ger-jr-icr.icr`.
* `readme.txt` — release notes, "Copyright Cloud Kayak Labs, Inc. 2026",
  version **12.01.981**.

---

## 7. PE resources and DFM forms

### 7.1 Inventory — `resources/`, 161 files

| Type | Count | Examples |
|---|---|---|
| 10 RCDATA | 61 | 35 form blobs, 20 `BB*` buttons, 3 `MSG_*`, `PLATFORMTARGETS`, `25371_0` |
| 6 Menu | 37 | — |
| 2 Bitmap | 23 | recognised as BMP by `file` |
| 12 Cursor | 14 | — |
| 1 Cursor | 14 | — |
| 3 Icon | 8 | — |
| 24 Manifest | 1 | `24_1` — readable XML |
| 16 Version | 1 | contains `12.01` and `981` (file/product version) |
| 14 | 1 | — |
| VCLSTYLE | 1 | `VCLSTYLE_WINDOWSDARK` |

Version string evidence: UTF-16 `12.01` at file `0x7B20A0` and full `12.01.981`
at file `0x7B5434`.

Section notes: `.rsrc` VA `0x66E000`, VirtualSize `0x363774` vs RawSize
`0x133DA3` (the difference explains zero-filled extractions), section entropy
**7.47**; the last section has entropy **8.00** (packed/compressed).
`unpacked/` payload has ~2.7 MB overlay beyond the PE image.

### 7.2 DFM forms — **not readable**

35 RCDATA form resources were extracted (`TABOUTSTORAGE`, `TBIGRAMDETAILFORM`,
`TBOOKFORMEX`, `TBUBBLESTG`, `TBUILDERSTG`, `TCHOOSEAPPLICATIONS`, `TCLOSEDLG`,
`TCOURSELISTVIEWSTG`, `TCOURSEVIEWSTG`, `TCUSTOMIZETESTFORM`, `TDEMOVIEWSTG`,
`TEVENTSSTG`, `TFRMBIGRAMHEATMAP`, `TGAMESSTG`, `TLESSONVIEWSTG`, …).

Findings:

* **15 of 35 are all-zero blobs** (32,768 bytes each): `TABOUTSTORAGE`,
  `TBIGRAMDETAILFORM`, `TBOOKFORMEX`, `TBUBBLESTG`, `TBUILDERSTG`,
  `TCHOOSEAPPLICATIONS`, `TCLOSEDLG`, `TCOURSELISTVIEWSTG`, `TCOURSEVIEWSTG`,
  `TCUSTOMIZETESTFORM`, `TDEMOVIEWSTG`, `TEVENTSSTG`, `TFRMBIGRAMHEATMAP`,
  `TGAMESSTG`, `TLESSONVIEWSTG`.
* `TLOGFORM` is near-zero (entropy 0.31).
* The remaining **19 have entropy 7.79–7.94** with only **30–41 % printable
  bytes**, i.e. they are compressed or otherwise transformed.
* **No** `TPF0` signature, **no** textual `object …: T…`, **no** zlib/gzip header
  and **no** raw-deflate stream at offsets 0–64.
* ASCII fragments *do* survive inside them: `VisAble`, `But@on2`, `Layout`,
  `Margin`, `Name`, `EFAULT_CwH`, `/# Curso`.

→ **Inference**: the payloads are a compressed/transformed DFM container using a
scheme not identifiable from these samples; the transform is **undetermined**
and this is stated as a limitation, not a conclusion. Consistently, all 35
`analysis/form_*.txt` files are **1 byte (empty)** — the forms were never decoded
by whatever tool produced `analysis/`.

**Consequence**: form layout, control positions and per-form published properties
cannot be read from this tree; class/component names instead come from
`analysis/classes.txt` (143) and `analysis/handlers.txt` (251).

---

## 8. Launcher scripts

### 8.1 `launcher.sh` — 10,413 bytes, Flatpak/Wine engine

Responsibilities, in order:

1. **Prefix**: `WINEPREFIX` default `${XDG_DATA_HOME:-$HOME/.local/share}/wine`;
   `WINEDEBUG=-all`; creates the prefix with `wineboot --init`.
2. **Package dirs**: `PACKAGED_DIR` = `$TYPINGMASTER_PACKAGED_DIR` →
   `/app/extra/typingmaster` → `/app/typingmaster`; `SHARE_DIR` likewise.
3. **Registry seed**: imports `wine-settings.reg`, guarded by a sha256-16 marker
   at `HKCU\Software\TypingMaster-Flatpak\SettingsVersion`.
4. **File sync**: uses `.package-build-id` and `.package-files` stamps;
   `comm -23` computes files present in the prefix but no longer shipped and
   **deletes** them; then copies changed files.
5. **DPI**: from `$TYPINGMASTER_DPI` or `~/.config/typingmaster.conf` `DPI=`
   (`auto|96..480`). `auto` ladder: screen height ≥1050 → 120, ≥1400 → 144,
   ≥2000 → 192. Applied with
   `wine reg add 'HKCU\Control Panel\Desktop' /v LogPixels …`.
6. **Reset**: `TYPINGMASTER_RESET_PREFIX=1` deletes the whole prefix.
7. **Defaults**: writes `%APPDATA%\TypingMaster11\<computer>.wk10` with
   `DpiAlertIsShown=1`, `FullScreen=0`.
8. **Launch**: `cd INSTALL_DIR; exec wine TypingMaster.exe "$@"`, where
   `INSTALL_DIR = $WINEPREFIX/drive_c/Program Files (x86)/TypingMaster`.

### 8.2 `run.sh` — 758 bytes

Syncs the repository's `app/` tree into the **Flatpak** prefix
`~/.var/app/com.typingmaster.TypingMaster/data/wine/drive_c/Program Files (x86)/TypingMaster`,
verifies the executable with `cmp`, then `exec flatpak run com.typingmaster.TypingMaster`.

### 8.3 `run-standalone.sh` — 907 bytes

Non-Flatpak path: sets `PATH` to `runtime/app-files/bin`, `LD_LIBRARY_PATH` to
the bundled runtime libs, `TYPINGMASTER_PACKAGED_DIR=$HERE/app`,
`TYPINGMASTER_SHARE_DIR=runtime/app-files/share/typingmaster`,
`TYPINGMASTER_WINEPREFIX=$HERE/prefix`, then `exec sh launcher.sh`.

→ **Inference**: two packaging targets (Flatpak and self-contained runtime) share
one Wine engine; data files are re-synced from `app/` on every start.

---

## 9. `native-re/functions.txt` and `native-re/analyze.err`

`functions.txt` (3 lines) is a rizin static listing of the **packed stub only**
(addresses `0x00DD…`, far from the `0x004…` Delphi image):

```
0x00dd3000  36  299 -> 147  entry0
0x00dd3340  119 1083 -> 609 fcn.00dd3340
0x00dd5eb9  26  331  -> 276 int.00dd5eb9
```

i.e. three functions, 1,431 bytes of code total — consistent with a small
decompression/dispatch loader plus one imported-trampoline entry.

`analyze.err` is **5 bytes**: only an ANSI clear-line sequence
(`ESC [ 2 K` + CR). Effectively empty — the analysis produced **no diagnostics**.

---

## 10. Evidence index

| Artifact | Role |
|---|---|
| `native-re/decompiled_typingmaster.c` | 12.2 MB / 502,553 lines / 13,137 functions |
| `native-re/functions.txt` | rizin stub listing (3 functions) |
| `native-re/analyze.err` | empty (ANSI clear only) |
| `analysis/classes.txt` | 143 classes |
| `analysis/handlers.txt` | 251 event handlers |
| `analysis/form_*.txt` (35) | all 1 byte — forms never decoded |
| `unpacked/code_401000.bin` | live image, 90,921 strings extracted |
| `app/` | 4,076 files / ~23 MB |
| `resources/` | 161 PE resource extractions |
| `/tmp/opencode/funs.txt` | `addr name size` for all 13,137 functions |
| `/tmp/opencode/xref_map.txt` | 2,130 function → string xrefs |
| `/tmp/opencode/methods2.txt`, `methods4.txt` | recovered method RTTI names |

---

## 11. Limitations and explicit inferences

1. **No signatures.** Ghidra recovered no function names, parameters or return
   types; everything in §4–§5 is anchored to string literals and RTTI records,
   not to typed code.
2. **33 % code coverage.** 67 % of the address range (including a 146,394-byte
   contiguous gap and the 14,261-byte product-key-dialog hole) is not decompiled.
3. **Licensing verification algorithm not isolated.** Grep over the code dump
   yields `license` → 0 hits as an identifier, `crypt` → 0, `md5` → 0, `sha` → 0,
   `productkey` → 0, `expire` → 0, `serial` → 1. No crypto/hashing routine could
   be tied to the `validate.php?key=` path. **Stated as a limitation; no
   circumvention was attempted or derived.**
4. **Exact WPM/arithmetic not shown.** Float constants were recovered
   (25/50/100/125) but the multiply/divide sequence is split across decompiler
   gaps; the formulas in §5.2 are therefore labelled inferences.
5. **DFM forms unreadable.** The transform over 19 high-entropy forms and the
   all-zero state of 16 others could not be identified (no zlib/gzip/deflate/`TPF0`).
6. **Inferences flagged in text** include: `%APPDATA%\TypingMaster11\TMLICEN.NET`
   as the runtime licence path; `TM11LICEN.NET` being a raw memory snapshot;
   `.hdr`/`.wzd` record semantics; the meaning of `TResultItem2` grade fields;
   KPM-vs-WPM labelling; and the `.exi`/`.exm` pairing.
7. **No file in the tree was modified.** This document is the only addition.
