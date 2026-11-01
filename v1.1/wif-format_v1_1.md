# WIF File Format Specification

**Version:** 1.1 · **Effective:** 1997 · **Author:** Shannon Young, describing the format by Ravi Nielsen and the 1996–97 WIF developer groups

This is a reference-style companion to the plain-language [WIF 1.1 guide](wif-format_v1_1_guide.md). It
covers the same format and the same scope — what the WeaveIt applications read and write —
laid out as tables for implementers. Where the two differ, the guide is authoritative.

---

### 1. WIF File (`.wif`)

A WIF file is plain text in the INI style: a sequence of `[SECTION]` headers, each followed by
`Key=Value` keylines. Section and key names are case-insensitive. A line whose first non-blank
character is `;` is a comment. All indices — ends, picks, shafts, treadles, colors — are
1-based.

#### 1.1 Conventions

| Topic | Rule |
| --- | --- |
| Encoding | Write UTF-8. Read UTF-8 first; fall back to ISO-8859-1. |
| Line endings | Write either. Read CRLF, LF, or a mix. |
| Comments | `;` at line start. Inline `;` after a number or bool is a comment; after a string it is literal text. |
| Case | Section and key names are case-insensitive. |
| Order | Sections, and keylines within a section, may appear in any order. |
| Duplicate keys | An error. Keep the first or the last; do not crash. |
| Empty section | Treat as absent. |
| Decimals | Period separator, e.g. `0.05`. |

#### 1.2 Data types

| Type | Notes |
| --- | --- |
| string | Free text to end of line. |
| integer | Whole number. Treat as 32-bit. |
| decimal | Real number, e.g. `0.05` or `24`. |
| bool | True: `true`, `on`, `yes`, `1`. False: `false`, `off`, `no`, `0`. Absent optional bool is false. |
| list | Comma-separated values of one type, e.g. `1,3,5`. Empty list and omitted keyline are equivalent. |

#### 1.3 Defaults and sparse data

Row sections (`[THREADING]`, `[TREADLING]`, `[LIFTPLAN]`, `[WARP COLORS]`, `[WEFT COLORS]`,
and the spacing and thickness sections) need not list every end or pick. A reader:

1. Creates `[WARP] Threads` ends and `[WEFT] Threads` picks.
2. Gives each the `[WARP]` / `[WEFT]` default `Color`, `Spacing`, and `Thickness`, and sets
   every shaft and treadle to `0` (unused).
3. Applies the keylines present in the file over those defaults.

A writer may omit any keyline equal to its default, or write them all.

#### 1.4 Header sections

**`[WIF]`** — Identifies the file. Required.

| Key | Type | Notes |
| --- | --- | --- |
| `Version` | string | Required. Always `1.1`. |
| `Source Program` | string | Required. Name of the writing program. |
| `Source Version` | string | Optional. Version of the writing program. |
| `Date` | string | Required. Free text; no dependable format. Parse leniently or display as text. |

**`[CONTENTS]`** — Table of contents. Required on write; advisory on read.

| Key | Type | Notes |
| --- | --- | --- |
| *section name* | bool | One keyline per section present, e.g. `WEAVING=1`. Only true entries are meaningful. |

Readers build the section list by scanning for headers, not from `[CONTENTS]`.

#### 1.5 Project sections

**`[TEXT]`** — Title and author. Optional.

| Key | Type | Notes |
| --- | --- | --- |
| `Title` | string | Optional. |
| `Author` | string | Optional. |

**`[NOTES]`** — Free-text notes. Optional.

| Key | Type | Notes |
| --- | --- | --- |
| *line number* | string | One keyline per line, from `1`. Reader joins with newlines. |

#### 1.6 Loom section

**`[WEAVING]`** — The loom. Required when the file carries a threading, tie-up, treadling, or
lift plan.

| Key | Type | Notes |
| --- | --- | --- |
| `Shafts` | integer | Required. > 0. |
| `Treadles` | integer | Required. > 0. Use the shaft count if the loom has no treadles. |
| `Rising Shed` | bool | Optional. `1` raises the selected shafts; `0` lowers them. Always write it; assume `1` if absent. |

#### 1.7 Warp and weft sections

**`[WARP]`** and **`[WEFT]`** — Thread count and defaults. Identical keys.

| Key | Type | Notes |
| --- | --- | --- |
| `Threads` | integer | Required. > 0. Number of ends (or picks). |
| `Color` | integer | Optional. Default color, as a `[COLOR TABLE]` index. Older files write `index,R,G,B`; read the first value. |
| `Units` | string | Required if `Spacing`, `Thickness`, or any spacing or thickness section is present. `Inches` or `Centimeters`. |
| `Spacing` | decimal | Optional. ≥ 0. Center-to-center distance, in `Units`. Equals 1 / sett. |
| `Thickness` | decimal | Optional. ≥ 0. Drawn yarn width, in `Units`. |

#### 1.8 Color sections

**`[COLOR PALETTE]`** — Size and channel range of the color table. Write whenever colors are
used.

| Key | Type | Notes |
| --- | --- | --- |
| `Entries` | integer | Required. > 0. Row count of `[COLOR TABLE]`. |
| `Range` | list | Required. `min,max` for each channel. Almost always `0,255`. |

**`[COLOR TABLE]`** — The colors.

| Key | Type | Notes |
| --- | --- | --- |
| *color index* | list | `R,G,B`, each within `Range`. Scale to 8-bit with `(v - min) * 255 / (max - min)`. |

**`[WARP COLORS]`** and **`[WEFT COLORS]`** — Per-end and per-pick color.

| Key | Type | Notes |
| --- | --- | --- |
| *end* / *pick* | integer | `[COLOR TABLE]` index. Absent → `[WARP]` / `[WEFT]` `Color`. Older files write `index,R,G,B`; read the first value. |

#### 1.9 Spacing and thickness sections

**`[WARP SPACING]`**, **`[WEFT SPACING]`**, **`[WARP THICKNESS]`**, **`[WEFT THICKNESS]`** —
Per-end and per-pick overrides.

| Key | Type | Notes |
| --- | --- | --- |
| *end* / *pick* | decimal | In `[WARP]` / `[WEFT]` `Units`. Absent → the `Spacing` or `Thickness` default. |

#### 1.10 Structure sections

A draft is defined by `[THREADING]` plus either `[TIEUP]` and `[TREADLING]`, or `[LIFTPLAN]`.
A file may carry all four.

**`[THREADING]`** — Shaft for each end.

```ini
[THREADING]
1=1      ;end 1 on shaft 1
2=2
3=3
4=4
```

Key is the end; value is a list of shafts. More than one shaft per end is legal but rare.
Absent end → shaft `0`, unthreaded.

**`[TIEUP]`** (with `[TREADLING]`) — Shafts tied to each treadle.

```ini
[TIEUP]
1=1,2    ;treadle 1 moves shafts 1 and 2
2=2,3
3=3,4
4=1,4
```

Key is the treadle; value is a list of shafts, each ≤ `[WEAVING] Shafts`. Value `0` or absent
→ treadle unused.

**`[TREADLING]`** (with `[TIEUP]`) — Treadles pressed for each pick.

```ini
[TREADLING]
1=1
2=2
3=3
4=4
```

Key is the pick; value is a list of treadles. More than one treadle is multi-treadling; store
as a list. Value `0` → no treadle pressed.

**`[LIFTPLAN]`** — Shafts raised for each pick, without treadles.

```ini
[LIFTPLAN]
1=1,2
2=2,3
3=3,4
4=1,4
```

Key is the pick; value is a list of shafts.

**Reading.** When `[TIEUP]`, `[TREADLING]`, and `[LIFTPLAN]` are all present, use the tie-up
and treadling. Fall back to the lift plan only if either is missing.

**Writing.** Write `[LIFTPLAN]` for every draft. For floor-loom drafts, also write `[TIEUP]`
and `[TREADLING]`.

---

### 2. Private sections

**`[PRIVATE <SourceID> <SectionName>]`** — Program-specific data outside the format.

| Field | Type | Notes |
| --- | --- | --- |
| `SourceID` | string | Short tag unique to the writing program. |
| `SectionName` | string | Section name within that program's namespace. |
| *keylines* | — | Any content. No line may begin with `[`. |

A reader skips any private section it does not recognize, resuming at the next line that
begins with `[`.

```ini
[PRIVATE WEAVEIT AWESOME]
Sparkles=1
```

---

### 3. Deprecated

Read only for compatibility. Never write.

| Item | Notes |
| --- | --- |
| `[WARP SYMBOL PALETTE]`, `[WEFT SYMBOL PALETTE]`, `[WARP SYMBOL TABLE]`, `[WEFT SYMBOL TABLE]`, `[WARP SYMBOLS]`, `[WEFT SYMBOLS]` | Symbols. Unused by current software. See the [guide](wif-format_v1_1_guide.md#symbols) for value forms. |
| `[WARP SPACING ZOOM]`, `[WARP THICKNESS ZOOM]`, `[WEFT SPACING ZOOM]`, `[WEFT THICKNESS ZOOM]`; `Spacing Zoom`, `Thickness Zoom` | Zoom. Not implemented. |
| `[BITMAP FILE]`, `[BITMAP IMAGE]`, `[BITMAP IMAGE DATA]` | Never implemented. Suspended at 1.1. |
| `[TRANSLATIONS]` | Suspended at 1.1. |
| `[DESIGN]` | Dropped at 1.1. |
| `[WIF] Developers` | Contact address. Privacy risk; ignore. |
| `[TEXT] Address`, `EMail`, `Telephone`, `FAX` | Personal contact data. Privacy risk; ignore. |
| `[WARP]` / `[WEFT]` `Colors`, `Palette`, `ColorMix` | Superseded by the color table. |
| `[WEAVING] Profile` | Obsolete. |
| `Units=Decipoints` | Treat as `Inches`. |

---

### 4. Integration notes

- **Platform file type.** On macOS and iOS, declare the shared UTI `org.wif-format.wif`. See
  [Sharing the `.wif` File Type on macOS and iOS](../apple-uti-setup.md).
- **Interoperability.** Files most often fail to open elsewhere because of a missing
  `[COLOR PALETTE]` / `[COLOR TABLE]`, a malformed `[CONTENTS]`, non-UTF-8 bytes, or a private
  section line beginning with `[`.
- **Shed direction.** `Rising Shed=0` inverts which face the drawdown shows.

---

### 5. Reference

- **Guide:** [wif-format_v1_1_guide.md](wif-format_v1_1_guide.md)
- **Example file:** [twill-color-and-weave.wif](twill-color-and-weave.wif)
- **Overview of all versions:** [README.md](../README.md)
- **Original WIF 1.1 text (historical):** [Copyright © 1996, 1997 Ravi Nielsen](http://www.tantradharma.com/maplehill/wif/wif1-1.txt)
- **Project site:** [wif-format.org](https://wif-format.org)
- **Discussion:** [github.com/WIF-Format/WIFStandard](https://github.com/WIF-Format/WIFStandard)

---

### Appendix A: Minimal WIF file

A 2-2 twill, straight-threaded on 4 shafts and 4 treadles, with alternating dark and light
ends and picks.

```ini
[WIF]
Version=1.1
Source Program=Example
Date=March 3, 1996

[CONTENTS]
COLOR PALETTE=1
COLOR TABLE=1
WEAVING=1
WARP=1
WEFT=1
THREADING=1
TIEUP=1
TREADLING=1
LIFTPLAN=1
WARP COLORS=1
WEFT COLORS=1

[COLOR PALETTE]
Entries=2
Range=0,255

[COLOR TABLE]
1=32,32,32
2=240,240,240

[WEAVING]
Shafts=4
Treadles=4
Rising Shed=1

[WARP]
Threads=8
Color=1
Units=Inches
Spacing=0.05

[WEFT]
Threads=8
Color=2
Units=Inches
Spacing=0.05

[THREADING]
1=1
2=2
3=3
4=4
5=1
6=2
7=3
8=4

[TIEUP]
1=1,2
2=2,3
3=3,4
4=1,4

[TREADLING]
1=1
2=2
3=3
4=4
5=1
6=2
7=3
8=4

[LIFTPLAN]
1=1,2
2=2,3
3=3,4
4=1,4
5=1,2
6=2,3
7=3,4
8=1,4

[WARP COLORS]
2=2
4=2
6=2
8=2

[WEFT COLORS]
1=1
3=1
5=1
7=1
```

---

Copyright © 2026 Shannon Young. Licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The WIF format itself is unowned;
see [License and attribution](../README.md#license-and-attribution).
