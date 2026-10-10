# WIF 1.2 — proposed additions and deprecations

> **This is an early draft for discussion, not a standard.** Nothing here is decided.
> It is not linked from the main WIF format guide and no software should implement it yet.

WIF 1.1 has not changed since 1997. This is the conservative path forward: a revision that
**stays in the INI text format**, formalizes the extensions that have already shipped in the field,
and officially retires the parts nothing implements. A companion proposal
([PR #1](https://github.com/WIF-Format/WIFStandard/pull/1), `proposals/wif2/`) explores a larger JSON
rewrite; the two are not exclusive.

## Principles

1. **Backward compatible both ways.** A 1.1 reader opens a 1.2 file and ignores what it does not
   recognise. A 1.2 reader opens a 1.1 file unchanged.
2. **Additive only.** No 1.1 section or keyline changes meaning. New things are new names.
3. **Standardize what exists.** Most additions here are the WeaveIt `PRIVATE` sections that already
   travel between the Mac, iOS, and web apps, promoted to public names.
4. **Retire the dead.** The sections the format guide already lists under *Obsolete and unsupported*
   become formally deprecated.

## Versioning

- A 1.2 file writes `[WIF] Version=1.2`. A writer that uses no 1.2 feature MAY keep `Version=1.1`.
- `Version` stays a display string; readers must not fail on an unexpected value.

---

## Additions

### A1. Text encoding

The 1997 text says "ASCII". Files in the wild are UTF-8, ISO-8859-1, or mojibake.

- A 1.2 file is **UTF-8**.
- It MAY declare it: `[WIF] Encoding=UTF-8`.
- A reader tries UTF-8 first and falls back to ISO-8859-1 (Latin-1). It writes UTF-8.

### A2. ISO dates

- `[WIF] Date` in a 1.2 file SHOULD be written `YYYY-MM-DD`.
- Readers stay lenient: display the value as text, and never reject a file over an unparseable date.

### A3. `[YARNS]` — a yarn table

The 1997 format models a yarn only as an RGB triplet in `[COLOR TABLE]`. `[YARNS]` promotes WeaveIt's
`[PRIVATE IWEAVEIT YARNS]` to a standard section that also carries fibre and size.

```
[YARNS]
; one entry per COLOR TABLE row, 1-based, same index space as [WARP COLORS] / [WEFT COLORS]
1=246,238,196, Yellow, cotton, 10/2
2=196,235,238, Blue, cotton, 5/2
```

- Row: `R, G, B, name, fiber, size`. `name`, `fiber`, `size` are free text; empty fields are allowed
  (write nothing between the commas, or the literal `NoValue` for compatibility with existing files).
- A 1.2 writer writes **both** `[COLOR TABLE]` (for 1.1 readers) and `[YARNS]`. The RGB values in the
  two sections must match.
- A 1.2 reader that sees `[YARNS]` uses it; the colours still come from `[COLOR TABLE]` if `[YARNS]`
  is absent.
- Per-thread spacing stays in `[WARP SPACING]` / `[WEFT SPACING]` — it is a property of the cloth, not
  the yarn.

### A4. `[WARP REPEATS]` / `[WEFT REPEATS]`

Promotes `[PRIVATE WEAVEIT WARP REPEATS]` / `[PRIVATE WEAVEIT WEFT REPEATS]`. Records the repeat
structure of a threading or treadling so it can be shown and edited as blocks.

```
[WARP REPEATS]
; N = location, repeatCount, length
1=0,3,7        ; from end 0, repeat a 7-end block 3 times
2=21,2,14
```

- Row: `location, repeatCount, length`. `location` is a 0-based index into the full (expanded) end or
  pick list.
- **Advisory.** The complete threading is always in `[THREADING]` etc.; this section is a hint about
  structure and may be ignored.

### A5. `[WEAVING] Weave Type=`

Promotes `[PRIVATE WEAVEIT BASICS] WEAVETYPE`. A hint for how to render and shade the drawdown.

```
[WEAVING]
Shafts=8
Treadles=8
Rising Shed=1
Weave Type=Balanced   ; Balanced | Warp-faced | Weft-faced | Double-weave | Profile | Double-width
```

Readers should also accept the legacy uppercase tokens (`BALANCE`, `WARPFACE`, `WEFTFACE`,
`DOUBLEWEAVE`, `PROFILEDRAFT`, `DOUBLEWIDTH`).

### A6. `[WEAVING] Loom Type=`

Records the loom mechanism, so a reader knows how to present the draft and which shed representation
is authoritative. A hint only — a 1.1 reader ignores the keyline and still draws the draft correctly
from `[THREADING]`, `[TIEUP]` / `[TREADLING]` / `[LIFTPLAN]`.

```
[WEAVING]
Shafts=8
Treadles=8
Rising Shed=1
Loom Type=Shaft   ; Shaft | Lift Plan | Rigid Heddle. Absent = Shaft.
```

**`Shaft`** (default) — a floor loom. Unchanged 1.1 behaviour: prefer `[TIEUP]` + `[TREADLING]`, fall
back to `[LIFTPLAN]`.

**`Lift Plan`** — a table loom, dobby, or software. `[LIFTPLAN]` is authoritative; any `[TIEUP]` /
`[TREADLING]` present is for reference only. This resolves the ambiguity the format guide documents
under *Tie-up and treadling vs lift plan*. (A floor-loom draft that also carries a lift plan for
compatibility keeps `Loom Type=Shaft`.)

**`Rigid Heddle`** — a rigid-heddle loom. The draft is still a valid 2-shaft WIF (or 4-shaft for a
2-heddle pickup setup): shaft 1 = heddle 1 holes, shaft 2 = heddle 1 slots, shaft 3 = heddle 2 holes,
shaft 4 = heddle 2 slots. Plain weave is the alternating `[THREADING]` `1,2,1,2…` and a `[LIFTPLAN]`
(or tie-up + treadling) that alternates raising shaft 1 and shaft 2 — exactly what a 1.1 rigid-heddle
file already contains. `Loom Type=Rigid Heddle` only tells a 1.2 reader to show holes and slots
instead of shafts, and heddle up / down instead of "raise shaft 1 / raise shaft 2".

```
[WEAVING]
Shafts=2
Treadles=2
Rising Shed=1
Loom Type=Rigid Heddle
Heddles=1              ; optional. 1 (default), or 2-3 for pickup and pattern heddles.
```

`Heddles=` is optional and only meaningful with `Loom Type=Rigid Heddle`.

### A7. `[WEAVING] Rising Shed=` is expected

Not new, but pinned down: a 1.2 file **always writes** `Rising Shed`. A reader that finds it missing
assumes `1` (rising / jack shed).

---

## Deprecations

A deprecated section or keyline is still **valid to read** — a reader ignores it silently — but a
1.2 writer **never emits** it.

### Sections

| Section | Why |
| --- | --- |
| `[WARP SYMBOL PALETTE]`, `[WEFT SYMBOL PALETTE]` | Symbol drawing; no current software uses it. |
| `[WARP SYMBOL TABLE]`, `[WEFT SYMBOL TABLE]` | " |
| `[WARP SYMBOLS]`, `[WEFT SYMBOLS]` | " |
| `[WARP SPACING ZOOM]`, `[WARP THICKNESS ZOOM]` | Variable-spacing magnification; never implemented. |
| `[WEFT SPACING ZOOM]`, `[WEFT THICKNESS ZOOM]` | " |
| `[BITMAP FILE]`, `[BITMAP IMAGE]`, `[BITMAP IMAGE DATA]` | Reserved in 1996, never built; suspended at 1.1. |
| `[TRANSLATIONS]` | Key-name localization; suspended at 1.1. |
| `[DESIGN]` | Dropped at 1.1. |

The **WIF Symbol data type** (`42` / `X` / `'X'` / `#219`) is deprecated with the symbol sections.

### Keylines

| Keyline | Why |
| --- | --- |
| `[WIF] Developers=` and any email keyline | Privacy — a shared file must not carry a contact address. |
| `[TEXT] Address=`, `EMail=`, `Telephone=`, `FAX=` | Privacy. Ignore on read; never write. |
| `[WARP]` / `[WEFT]` `Colors=`, `Palette=`, `ColorMix=` | Superseded by the colour table (already obsolete in 1.1). |
| `[WARP]` / `[WEFT]` `Symbol=`, `Symbol Number=`, `Spacing Zoom=`, `Thickness Zoom=` | Go with the deprecated symbol / zoom sections. |
| `[WEAVING] Profile=` | Obsolete constant. |
| `Units=Decipoints` | Never appears in real files; use `Inches` or `Centimeters`. |

### `[CONTENTS]` becomes advisory

The 1997 spec requires `[CONTENTS]` and requires it to list every present section. In practice it is
frequently wrong, and the format guide already tells readers not to trust it.

- **Readers** MUST build their section list by scanning for `[SECTION]` headers. `[CONTENTS]` is a
  hint only. A present-but-empty section counts as absent.
- **Writers** MAY still emit `[CONTENTS]` (1.1 readers may want it); if they do, it must be correct —
  one `NAME=1` keyline per line for each present section.

This is a behaviour change for readers, not a removal.

---

## Clarifications

Not normative changes — ambiguities in the 1997 text, pinned to match the format guide.

- **Default values and sparse data.** A reader creates `[WARP].Threads` ends and `[WEFT].Threads`
  picks with the `[WARP]` / `[WEFT]` defaults and shaft/treadle `0`, then applies the keylines that
  exist. Row sections need not list every index or list them in order.
- **Multi-treadling.** A `[TREADLING]` keyline may name more than one treadle; store it as a list.
- **Multi-shaft threading.** `1=1,5` (one end through two shafts) is legal. A reader that does not
  support it takes the first shaft.
- **Line endings.** Accept CRLF and LF, mixed within one file.
- **Duplicate keylines** in one section are an error; a reader keeps one and does not crash.

---

## Open questions

- **`[YARNS]` field set.** Is `R,G,B,name,fiber,size` right? Should `fiber` and `size` be one free
  field (as WeaveIt's `yarnType` is today, `"5/2 cotton"`) or two? Any need for ply / wraps-per-inch?
- **`[YARNS]` vs `[COLOR TABLE]`.** Requiring writers to emit both keeps 1.1 compatibility but
  duplicates the RGB values. Acceptable, or should `[YARNS]` be able to stand alone with a
  `Version>=1.2` gate?
- **A `[PROJECT]` section?** WeaveIt keeps planning data (loom waste, take-up, shrinkage, finished
  width/length, target sett/ppi) in `[PRIVATE IWEAVEIT PROJECT]`. Standardize it, or leave it private
  because the assumptions behind those numbers are app-specific? (Leaning: leave it private.)
- **Deprecating `Decipoints`.** Safe, or is there a program that relies on it?
- **`Weave Type` values.** Readable names vs. keeping the legacy uppercase tokens as canonical.
- **Encoding keyline.** Is `[WIF] Encoding=` worth adding, or is "1.2 means UTF-8" enough?
- **Rigid-heddle sheds.** Mapping heddle up / down to "raise shaft 1 / raise shaft 2" works for plain
  weave. Is that enough for 2- and 3-heddle pattern work, or does a rigid-heddle draft need a
  `neutral` shed and a way to record picked-up ends?

---

## License

Copyright © 2026 Shannon Young. Licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), like the rest of this repository. Nothing
here claims any right in the WIF format itself.
