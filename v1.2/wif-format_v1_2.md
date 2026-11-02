# WIF File Format Specification

**Version:** 1.2 (draft proposal) · **Effective:** not adopted · **Author:** Shannon Young, extending the format by Ravi Nielsen and the 1996–97 WIF developer groups

> **Draft proposal, not a standard.** No software should implement it yet. Rationale and open
> questions: [proposal](proposal.md). Plain-language walkthrough: [guide](wif-format_v1_2_guide.md).

WIF 1.2 is additive over 1.1. Every 1.1 file is a valid 1.2 file, and no 1.1 section or key
changes meaning. Items new or changed in 1.2 are marked **(1.2)**.

---

### 1. WIF File (`.wif`)

A WIF file is UTF-8 plain text in the INI style: a sequence of `[SECTION]` headers, each
followed by `Key=Value` keylines. Section and key names are case-insensitive. A line whose
first non-blank character is `;` is a comment. All indices are 1-based except the `location`
field of the repeat sections.

#### 1.1 Conventions

| Topic | Rule |
| --- | --- |
| Encoding | **(1.2)** Write UTF-8. Read UTF-8 first; fall back to ISO-8859-1. |
| Line endings | Write either. Read CRLF, LF, or a mix. |
| Comments | `;` at line start. Inline `;` after a number or bool is a comment; after a string it is literal text. |
| Case | Section and key names are case-insensitive. |
| Order | Sections, and keylines within a section, may appear in any order. |
| Duplicate keys | An error. Keep one; do not crash. |
| Section discovery | **(1.2)** Scan for `[SECTION]` headers. `[CONTENTS]` is advisory. |
| Empty section | Treat as absent. |
| Unknown sections and keys | Ignore. |
| Decimals | Period separator, e.g. `0.05`. |

#### 1.2 Data types

| Type | Notes |
| --- | --- |
| string | Free text to end of line. UTF-8. |
| integer | Whole number. Treat as 32-bit. |
| decimal | Real number, e.g. `0.05` or `24`. |
| bool | True: `true`, `on`, `yes`, `1`. False: `false`, `off`, `no`, `0`. Absent optional bool is false. |
| list | Comma-separated values, e.g. `1,3,5`. Empty list and omitted keyline are equivalent. |
| date | **(1.2)** `YYYY-MM-DD` on write. Read leniently; never reject. |

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
| `Version` | string | Required. `1.2`. A writer using no 1.2 feature may write `1.1`. Never reject on value. |
| `Source Program` | string | Required. Name of the writing program. |
| `Source Version` | string | Optional. Version of the writing program. |
| `Date` | date | Required. **(1.2)** `YYYY-MM-DD`. |
| `Encoding` | string | **(1.2)** Optional. `UTF-8`. |

**`[CONTENTS]`** — Table of contents. **(1.2)** Optional; advisory.

| Key | Type | Notes |
| --- | --- | --- |
| *section name* | bool | One keyline per section present, e.g. `WEAVING=1`. If written, must be complete and correct. |

#### 1.5 Project sections

**`[TEXT]`** — Title and author. Optional.

| Key | Type | Notes |
| --- | --- | --- |
| `Title` | string | Optional. |
| `Author` | string | Optional. Name only. |

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
| `Rising Shed` | bool | **(1.2)** Always written. `1` raises the selected shafts; `0` lowers them. Absent → `1`. |
| `Loom Type` | string | **(1.2)** Optional. `Shaft`, `Lift Plan`, or `Rigid Heddle`. Absent → `Shaft`. See 1.6.1. |
| `Heddles` | integer | **(1.2)** Optional. `Loom Type=Rigid Heddle` only. `1`–`3`. Absent → `1`. |
| `Weave Type` | string | **(1.2)** Optional. Rendering hint. See 1.6.2. |

##### 1.6.1 Loom Type

| Value | Authoritative shed data | Notes |
| --- | --- | --- |
| `Shaft` | `[TIEUP]` + `[TREADLING]` | Fall back to `[LIFTPLAN]` if either is missing. |
| `Lift Plan` | `[LIFTPLAN]` | Any `[TIEUP]` / `[TREADLING]` is reference only. |
| `Rigid Heddle` | As `Shaft` | Shafts map to heddle positions as below. |

Rigid-heddle shaft mapping:

| Shaft | Meaning |
| --- | --- |
| 1 | Heddle 1, holes. |
| 2 | Heddle 1, slots. |
| 3 | Heddle 2, holes. |
| 4 | Heddle 2, slots. |
| 5, 6 | Heddle 3, holes and slots. |

`Shafts` equals 2 × `Heddles`.

##### 1.6.2 Weave Type

| Value | Legacy token (accept on read) |
| --- | --- |
| `Balanced` | `BALANCE` |
| `Warp-faced` | `WARPFACE` |
| `Weft-faced` | `WEFTFACE` |
| `Double-weave` | `DOUBLEWEAVE` |
| `Profile` | `PROFILEDRAFT` |
| `Double-width` | `DOUBLEWIDTH` |

#### 1.7 Warp and weft sections

**`[WARP]`** and **`[WEFT]`** — Thread count and defaults. Identical keys.

| Key | Type | Notes |
| --- | --- | --- |
| `Threads` | integer | Required. > 0. Number of ends (or picks). |
| `Color` | integer | Optional. Default color, as a `[COLOR TABLE]` index. Older files write `index,R,G,B`; read the first value. |
| `Units` | string | Required if `Spacing`, `Thickness`, or any spacing or thickness section is present. `Inches` or `Centimeters`. |
| `Spacing` | decimal | Optional. ≥ 0. Center-to-center distance, in `Units`. Equals 1 / sett. |
| `Thickness` | decimal | Optional. ≥ 0. Drawn yarn width, in `Units`. |

#### 1.8 Color and yarn sections

**`[COLOR PALETTE]`** — Size and channel range of the color table. Write whenever colors are
used.

| Key | Type | Notes |
| --- | --- | --- |
| `Entries` | integer | Required. > 0. Row count of `[COLOR TABLE]`. |
| `Range` | list | Required. `min,max` for each channel. Almost always `0,255`. |

**`[COLOR TABLE]`** — The colors. Required whenever `[YARNS]` is present.

| Key | Type | Notes |
| --- | --- | --- |
| *color index* | list | `R,G,B`, each within `Range`. Scale to 8-bit with `(v - min) * 255 / (max - min)`. |

**`[YARNS]`** — **(1.2)** Yarn table. Optional.

| Field | Type | Notes |
| --- | --- | --- |
| *key* | integer | Color index. Same index space as `[COLOR TABLE]`. |
| `R`, `G`, `B` | integer | Must equal the `[COLOR TABLE]` row. |
| `name` | string | Optional. Empty or `NoValue` → none. |
| `fiber` | string | Optional. Empty or `NoValue` → none. |
| `size` | string | Optional. Free text, e.g. `10/2`. Empty or `NoValue` → none. |

```ini
[YARNS]
1=246,238,196, Yellow, cotton, 10/2
2=196,235,238, Blue, cotton, 5/2
```

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
A file may carry all four; `Loom Type` selects which is authoritative (1.6.1).

**`[THREADING]`** — Shaft for each end.

```ini
[THREADING]
1=1      ;end 1 on shaft 1
2=2
3=3
4=4
```

Key is the end; value is a list of shafts. More than one shaft is legal; a reader that does
not support it takes the first. Absent end → shaft `0`, unthreaded.

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

**Writing.** Write `[LIFTPLAN]` for every draft. For `Loom Type=Shaft`, also write `[TIEUP]`
and `[TREADLING]`.

#### 1.11 Repeat sections — (1.2)

**`[WARP REPEATS]`** and **`[WEFT REPEATS]`** — Repeat structure of the threading or
treadling. Optional; advisory.

| Field | Type | Notes |
| --- | --- | --- |
| *key* | integer | Row number, from `1`. |
| `location` | integer | **0-based** index of the block's first end (or pick) in the expanded sequence. |
| `repeatCount` | integer | Times the block is woven. |
| `length` | integer | Ends (or picks) in one block. |

```ini
[WARP REPEATS]
1=0,3,7        ;from the first end, a 7-end block three times
2=21,2,14
```

The expanded threading and treadling are always written in full in the structure sections. A
reader may ignore the repeat sections.

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

**(1.2)** A writer that emits `[YARNS]`, `[WARP REPEATS]`, `[WEFT REPEATS]`, or
`[WEAVING] Weave Type` should stop emitting the private sections they replace:

| Private section | Replaced by |
| --- | --- |
| `[PRIVATE IWEAVEIT YARNS]` | `[YARNS]` |
| `[PRIVATE WEAVEIT WARP REPEATS]` | `[WARP REPEATS]` |
| `[PRIVATE WEAVEIT WEFT REPEATS]` | `[WEFT REPEATS]` |
| `[PRIVATE WEAVEIT BASICS] WEAVETYPE` | `[WEAVING] Weave Type` |

---

### 3. Deprecated

Valid to read; ignore silently. A 1.2 writer never emits these.

| Item | Notes |
| --- | --- |
| `[WARP SYMBOL PALETTE]`, `[WEFT SYMBOL PALETTE]`, `[WARP SYMBOL TABLE]`, `[WEFT SYMBOL TABLE]`, `[WARP SYMBOLS]`, `[WEFT SYMBOLS]` | Symbols, and the WIF Symbol data type. |
| `[WARP SPACING ZOOM]`, `[WARP THICKNESS ZOOM]`, `[WEFT SPACING ZOOM]`, `[WEFT THICKNESS ZOOM]` | Zoom. Never implemented. |
| `[BITMAP FILE]`, `[BITMAP IMAGE]`, `[BITMAP IMAGE DATA]` | Never implemented. Suspended at 1.1. |
| `[TRANSLATIONS]` | Suspended at 1.1. |
| `[DESIGN]` | Dropped at 1.1. |
| `[WIF] Developers` | Contact address. Privacy risk. |
| `[TEXT] Address`, `EMail`, `Telephone`, `FAX` | Personal contact data. Privacy risk. |
| `[WARP]` / `[WEFT]` `Colors`, `Palette`, `ColorMix` | Superseded by the color table. |
| `[WARP]` / `[WEFT]` `Symbol`, `Symbol Number`, `Spacing Zoom`, `Thickness Zoom` | With the symbol and zoom sections. |
| `[WEAVING] Profile` | Obsolete. |
| `Units=Decipoints` | Read as `Inches`. |

---

### 4. Integration notes

- **Backward compatibility.** A 1.1 reader ignores `[YARNS]`, the repeat sections, and the new
  `[WEAVING]` keys, and still reads the complete draft. Keep `[COLOR TABLE]` and the expanded
  structure sections for that reason.
- **Strict 1.1 readers.** Some refuse a file without `[CONTENTS]`. Write it if you target them.
- **Platform file type.** Unchanged: `.wif`, UTI `org.wif-format.wif`. See
  [Sharing the `.wif` File Type on macOS and iOS](../apple-uti-setup.md).

---

### 5. Reference

- **Proposal and open questions:** [proposal.md](proposal.md)
- **Guide:** [wif-format_v1_2_guide.md](wif-format_v1_2_guide.md)
- **Example files:** [twill-color-and-weave.wif](twill-color-and-weave.wif), [rigid-heddle-plain.wif](rigid-heddle-plain.wif)
- **WIF 1.1:** [v1.1](../v1.1/)
- **Overview of all versions:** [README.md](../README.md)
- **Original WIF 1.1 text (historical):** [Copyright © 1996, 1997 Ravi Nielsen](http://www.tantradharma.com/maplehill/wif/wif1-1.txt)
- **Discussion:** [PR #2](https://github.com/WIF-Format/WIFStandard/pull/2)

---

### Appendix A: Minimal WIF 1.2 file

A single-heddle rigid-heddle plain weave, 8 ends by 4 picks, with a yarn table. The same file
is [`rigid-heddle-plain.wif`](rigid-heddle-plain.wif).

```ini
[WIF]
Version=1.2
Source Program=Example
Date=2026-10-06
Encoding=UTF-8

[COLOR PALETTE]
Entries=2
Range=0,255

[COLOR TABLE]
1=168,0,0
2=231,224,85

[YARNS]
1=168,0,0, Barn red, cotton, 8/2
2=231,224,85, Gold, cotton, 8/2

[WEAVING]
Shafts=2
Treadles=2
Rising Shed=1
Loom Type=Rigid Heddle
Heddles=1
Weave Type=Balanced

[WARP]
Threads=8
Color=1
Units=Inches
Spacing=0.0833

[WEFT]
Threads=4
Color=2
Units=Inches
Spacing=0.0833

[THREADING]
1=1
2=2
3=1
4=2
5=1
6=2
7=1
8=2

[WARP REPEATS]
1=0,4,2

[LIFTPLAN]
1=1
2=2
3=1
4=2

[TIEUP]
1=1
2=2

[TREADLING]
1=1
2=2
3=1
4=2
```

---

Copyright © 2026 Shannon Young. Licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The WIF format itself is unowned;
see [License and attribution](../README.md#license-and-attribution).
