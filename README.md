# Weaving Information File (WIF)

The **Weaving Information File** format was created in 1996 and revised in 1997 so that
weaving programs could exchange drafts with one another. It is a plain-text format, based
on the [INI file](https://en.wikipedia.org/wiki/INI_file) convention, and it was designed
to be read by a person as easily as by a program.

This document is a plain-language description of the format as it is actually used today,
written for two readers at once: a **weaver**, who knows what a threading or a tie-up is
but not how it is stored, and a **developer**, who needs to read or write the bytes
correctly. Each section below first says what the data means at the loom, then — under
**In the file** — exactly how it is encoded.

It is not the original specification and does not reproduce it. The original WIF 1.1 text,
[Copyright © 1996, 1997 Ravi Nielsen](http://www.tantradharma.com/maplehill/wif/wif1-1.txt),
remains the historical source.

### Scope

This guide limits itself to what the [Canyon Art WeaveIt](https://weaveit.com/) applications read and write. Their author was
one of the original WIF 1.1 developers, so where those applications ignore a part of the
format, this guide treats that part as **effectively deprecated** and describes it only
briefly, under [Obsolete and unsupported](#obsolete-and-unsupported).

### Credits

WIF was written by Ravi Nielsen (Maple Hill Software). The 1.0 and 1.1 developer groups
included Bob Keates (Fiberworks), Rob Sinkler (Swiftweave), Dana Cartwright (WeaveMaker),
Bjørn Myhre (WeavePoint), Jane Eisenstein, Sally Breckenridge (WeaveIt), Mark Kloosterman
and Dini Cameron (ProWeave), Mitch Eatough, and Peter Straus (AVL).

---

## Terminology

### At the loom

| Term | Meaning |
| --- | --- |
| **warp** | The threads held under tension on the loom, running the length of the cloth. |
| **weft** | The threads carried across the warp, one row at a time. |
| **end** | A single warp thread. |
| **pick** | A single weft thread — one row of weaving. |
| **shaft** | A frame that raises or lowers a group of ends. Also called a *harness*. |
| **heddle** | A loop on a shaft that one end passes through. |
| **treadle** | A foot pedal on a floor loom. Pressing it moves the shafts tied to it. |
| **threading** | Which shaft each end passes through, read across the warp. |
| **tie-up** | Which shafts each treadle is connected to. |
| **treadling** | The order in which the treadles are pressed, one entry per pick. |
| **lift plan** | The shafts raised for each pick, given directly — used instead of a tie-up and treadling. |
| **rising / sinking shed** | Whether the loom raises the selected shafts (jack / rising shed) or lowers them (countermarch, counterbalance / sinking shed). |
| **sett** | How closely the ends are spaced, usually in ends per inch (EPI). Picks per inch (PPI) is the weft equivalent. |
| **drawdown** | The picture of the finished cloth produced by combining threading, tie-up / lift plan, and treadling. |

### In the file

| Term | Meaning |
| --- | --- |
| **section** | A named block, written `[SECTION NAME]`, containing keylines. |
| **keyline** | One `Key=Value` line inside a section. |
| **map** | A section whose keys are either integers (`1`, `2`, `3`, …) or names. WIF sections are all maps. |
| **array** | A comma-separated list of values in a single keyline, e.g. `1,3,5`. |

---

## Data types

| Type | Notes |
| --- | --- |
| **Integer** | A whole number. Treat it as 32-bit. |
| **Real** | A decimal number, e.g. `0.05` or `24`. The 1997 text calls this *real*; some call it *double*. |
| **String** | Text. The 1997 text says ASCII; in practice see [Text encoding](#text-encoding). |
| **Boolean** | True is any of `true`, `on`, `yes`, `1`; false is any of `false`, `off`, `no`, `0`. An absent optional boolean is false. |
| **Array** | A comma-separated list of one of the above in a single keyline. An empty list and an omitted keyline mean the same thing. |

---

## File structure

A WIF file is a sequence of sections. Each section starts with its name in brackets on its
own line and is followed by keylines until the next section starts.

```
[SECTION NAME]
Key=Value
1=Value
2=Value
```

- **Comments.** A line whose first non-blank character is `;` is a comment. A `;` may also
  follow a number or boolean value to comment it. A `;` is **not** a comment after a string
  value — there it is literal text.
- **Case.** Section and key names are case-insensitive. `[WEAVING]`, `[Weaving]`, and
  `Units=Inches` vs `Units=inches` are all equivalent.
- **Order.** Sections may appear in any order, and keylines within a section may appear in
  any order. Do not assume `[THREADING]` is written `1=`, `2=`, `3=` … in sequence.
- **Duplicate keys.** Writing the same key twice in one section is an error. A reader may
  keep the first or the last; it should not crash.
- **Line endings.** Files use CRLF or LF, and sometimes both in one file. Accept any.

### Default values and sparse data

This is the single most common source of confusion.

The row sections — `[THREADING]`, `[TREADLING]`, `[WARP COLORS]`, `[WARP SPACING]`, and so
on — do **not** have to list every end or pick. A reader is expected to:

1. Create `[WARP].Threads` ends and `[WEFT].Threads` picks up front.
2. Give each one the default from `[WARP]` / `[WEFT]` — the default `Color`, `Spacing`,
   and `Thickness` — and set every shaft and treadle to `0`, meaning *unused*.
3. Then apply whatever keylines the file does contain, overriding those defaults.

So a `[WARP COLORS]` section that contains only `3=2` means *every end is the default warp
color except end 3, which is color 2*. A writer is allowed to omit any keyline whose value
already equals the default. A writer may also choose to write every keyline; both are valid.

### Text encoding

The 1997 text predates Unicode and says "ASCII". Real files are a mix: UTF-8, ISO-8859-1
(Latin-1), and files that have been decoded and re-encoded until the punctuation is
garbled. A robust reader should try UTF-8 first and fall back to Latin-1. Write UTF-8.

---

## Required sections

### WIF

Identifies the file as a WIF and records which program wrote it.

**In the file** — required. String keylines.

```
[WIF]
Version=1.1                 ;required. Always 1.1.
Source Program=Mac WeaveIt  ;required. The program that wrote the file.
Source Version=1.0          ;optional. That program's version.
Date=October 6, 2016        ;required. Free text — see the note below.
```

_The 1997 text also lists a `Developers` keyline holding a contact email address, and
marks it required. **Do not write it.** A WIF file travels through forums and email
attachments; an address embedded in one is a privacy and spam risk to whoever put it
there, and no reader needs it. Omit the keyline; if some strict reader forces the issue,
use a shared project address, never a person's. Readers should ignore it. `Date` has no
dependable format — see [Notes for implementers](#the-date-keyline-is-free-text)._

### CONTENTS

A table of contents listing the other sections in the file.

**In the file** — required by the 1997 text. Boolean keylines keyed by section name; only
`true` entries are meaningful. Treat it as a **hint, not the source of truth**: scan the
file for actual `[SECTION]` headers instead (see
[Notes for implementers](#dont-trust-contents)).

```
[CONTENTS]
WEAVING=1
WARP=1
WEFT=1
THREADING=1
TIEUP=1
TREADLING=1
LIFTPLAN=0
```

---

## Project sections

### TEXT

The title and author of the draft.

**In the file** — optional. String keylines.

```
[TEXT]
Title=Point twill, color & weave  ;optional
Author=Sally Breckenridge          ;optional
```

_The 1997 text also defines `Address`, `EMail`, `Telephone`, and `FAX` keylines. **Never
write them, and ignore them on read** — they are personal contact information that does not
belong in a file meant to be shared._

### NOTES

Free-text notes from the person who made the draft.

**In the file** — optional. A map from line number to text, one keyline per line. Values
contain no line breaks; a reader joins the lines with newlines.

```
[NOTES]
1=This is a 2-2 twill double cloth. It is woven top-bottom-bottom-top.
2=Change the colors to create your own twill double cloth.
3=Use color & weave to create a sequence.
```

---

## The loom

### WEAVING

Describes the loom: how many shafts and treadles it has, and whether it is a rising-shed
loom. This section links the warp and the weft together, and is required whenever the file
carries a threading, tie-up, treadling, or lift plan. A file used only to share yarn
colors may still include it.

**In the file**

```
[WEAVING]
Shafts=8       ;required (Integer > 0)
Treadles=8     ;required (Integer > 0). Use the shaft count if the loom has no treadles.
Rising Shed=1  ;optional (Boolean)
```

`Rising Shed=1` means the loom raises the shafts named in the tie-up or lift plan (a jack
loom); `0` means it lowers them (countermarch or counterbalance), which inverts the
drawdown. The 1997 text makes this optional and therefore false-by-default, but most
software assumes a rising shed. **Write it explicitly.**

---

## The warp and the weft

### WARP and WEFT

The setup for the warp and the weft as a whole: how many ends or picks there are, their
default color, and their default spacing and thickness. Individual ends and picks can
override these in the row sections below.

Both sections have the same keylines.

**In the file**

```
[WARP]         (and, identically, [WEFT])
Threads=8      ;required (Integer > 0). The number of ends (or picks).
Color=1        ;optional. Default color as an index into the COLOR TABLE.
Units=Inches   ;required if Spacing or Thickness is present.
               ;one of Decipoints, Inches, Centimeters.
Spacing=0.05   ;optional (Real >= 0). Center-to-center distance between ends, in Units
               ;— i.e. 1 / sett. 0.05 in = 20 ends per inch.
Thickness=0.05 ;optional (Real >= 0). Visual width of the yarn, in Units.
```

- `Color` may appear as a bare index (`1`) or, in older files, as `index,R,G,B`. Read the
  first number; ignore any trailing RGB. Decode it as an array so old files still load.
- WeaveIt treats `Decipoints` as `Inches`.

---

## Threading and drawdown

These four sections describe how the cloth is actually woven. A file provides the
threading plus **either** a tie-up and treadling **or** a lift plan.

### THREADING

Which shaft each warp end passes through, read across the warp from one edge.

**In the file** — a map from end number to shaft number(s).

```
[THREADING]
1=1     ;end 1 on shaft 1
2=2
3=3
4=4
5=1
```

- `1=1,5` means end 1 is threaded through heddles on **both** shaft 1 and shaft 5. This is
  rare; WeaveIt reads a single shaft per end.
- An end with no keyline is on shaft `0` — not threaded (see
  [Default values and sparse data](#default-values-and-sparse-data)).

### TIEUP

On a floor loom, which shafts each treadle is tied to. Pressing that treadle moves exactly
those shafts.

**In the file** — a map from treadle number to an array of shaft numbers. The largest
shaft number is `[WEAVING].Shafts`.

```
[TIEUP]
1=1,2   ;treadle 1 moves shafts 1 and 2
2=2,3
3=3,4
4=1,4
```

- A treadle with the value `0`, or with no keyline, is unused.
- `[TIEUP]` is used together with `[TREADLING]`. A file uses this pair **or** `[LIFTPLAN]`.

### TREADLING

The order the treadles are pressed, one entry per pick.

**In the file** — a map from pick number to an array of treadle numbers.

```
[TREADLING]
1=1     ;pick 1: press treadle 1
2=2
3=3
4=4
```

- More than one treadle in a keyline (`5=1,3`) is *multi-treadling* — the shafts from all
  listed treadles move together.
- A value of `0` is a pick with no treadle pressed.

### LIFTPLAN

The shafts raised for each pick, given directly — the same information a tie-up and
treadling produce, but without reference to treadles. Table looms, dobby looms, and
weaving software work from a lift plan.

**In the file** — a map from pick number to an array of shaft numbers.

```
[LIFTPLAN]
1=1,2
2=2,3
3=3,4
4=1,4
```

A file may legitimately carry **both** a lift plan and a tie-up plus treadling — the
WeaveIt applications write the lift plan on every draft, because some programs read only
the lift plan and ignore the tie-up, and add the tie-up and treadling for floor-loom
drafts. When a reader sees all three, it should use the tie-up and treadling (they record
the loom's actual tie-up) and fall back to the lift plan only if one of them is missing.

---

## Color

### COLOR PALETTE and COLOR TABLE

The set of yarn colors used in the draft. `[COLOR PALETTE]` says how many colors there are
and what number range the color channels use; `[COLOR TABLE]` lists the colors.

**In the file**

```
[COLOR PALETTE]
Entries=2     ;required (Integer > 0). Number of rows in COLOR TABLE.
Range=0,255   ;required (Integer,Integer). Min and max value for each R, G, B channel.

[COLOR TABLE]
1=32,32,32    ;color 1 as R,G,B, each within Range
2=240,240,240
```

- `Range` is almost always `0,255`. If it is not, scale a channel to 8-bit with
  `(value - min) * 255 / (max - min)`. For example, with `Range=0,999`, color `999,999,999`
  is white and `0,0,0` is black.
- **Include these sections.** Many programs refuse to open a file that has colored threads
  but no color table.

### WARP COLORS and WEFT COLORS

The color of each individual end or pick, as an index into the color table. With the
threading, this is what produces stripes, plaids, and color-and-weave effects.

**In the file** — a map from end (or pick) number to a color table index.

```
[WARP COLORS]        (and, identically, [WEFT COLORS])
2=2     ;end 2 uses color 2; all other ends use [WARP].Color
4=2
6=2
8=2
```

- An end or pick with no keyline uses the default `Color` from `[WARP]` / `[WEFT]`.
- Older files may write `index,R,G,B`. Read the first number; ignore the RGB.

---

## Spacing and thickness

### WARP SPACING, WARP THICKNESS, WEFT SPACING, WEFT THICKNESS

Per-end and per-pick overrides of the loom-wide `Spacing` and `Thickness` from `[WARP]`
and `[WEFT]` — for drafts that mix thick and thin yarns, or that change the sett partway
across.

**In the file** — a map from end (or pick) number to a `Real` value, measured in the
`Units` declared in `[WARP]` / `[WEFT]`. If any of these sections is present, `Units` is
required.

```
[WARP SPACING]       (and, identically, [WEFT SPACING])
5=0.10  ;end 5 sits in a 0.10 in space; other ends use [WARP].Spacing
6=0.10

[WARP THICKNESS]     (and, identically, [WEFT THICKNESS])
5=0.08  ;end 5's yarn draws 0.08 in wide; other ends use [WARP].Thickness
6=0.08
```

An end or pick with no keyline uses the `[WARP]` / `[WEFT]` default.

---

## Private sections

Room for a program to store data that WIF does not model, without breaking other programs.

**In the file** — the section name is `[PRIVATE <SourceID> <SectionName>]`, where
`<SourceID>` is a short tag unique to the program that wrote it. A program that does not
recognize a private section skips it. **No line inside a private section may begin with
`[`** — that is how a reader finds where the next real section starts.

```
[PRIVATE WEAVEIT AWESOME]
Sparkles=1
```

A reader that does not know `WEAVEIT` skips straight from the `[PRIVATE WEAVEIT AWESOME]`
header to the next `[` at the start of a line.

---

## A complete example

A small 2-2 twill, threaded straight on 4 shafts, with alternating dark and light in both
the warp and the weft (a color-and-weave setup). The color sections are written sparsely:
the warp defaults to dark and only the light ends are listed; the weft defaults to light
and only the dark picks are listed.

```
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
WARP COLORS=1
WEFT COLORS=1

[COLOR PALETTE]
Entries=2
Range=0,255

[COLOR TABLE]
1=32,32,32       ;dark
2=240,240,240    ;light

[WEAVING]
Shafts=4
Treadles=4
Rising Shed=1

[WARP]
Threads=8
Color=1          ;default: dark
Units=Inches
Spacing=0.05     ;20 ends per inch

[WEFT]
Threads=8
Color=2          ;default: light
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
1=1,2            ;a 2-2 twill: each treadle lifts two adjacent shafts
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

[WARP COLORS]
2=2             ;ends 2, 4, 6, 8 are light; the rest stay dark
4=2
6=2
8=2

[WEFT COLORS]
1=1             ;picks 1, 3, 5, 7 are dark; the rest stay light
3=1
5=1
7=1
```

Reading this file: for pick 1 the weaver presses treadle 1, which raises shafts 1 and 2.
With the straight threading, that is warp ends 1, 2, 5, and 6 — they lie on top of pick 1,
while ends 3, 4, 7, and 8 pass under it. Pick 2 presses treadle 2 (shafts 2 and 3), and so
on, walking the raised pair across the warp to draw the twill diagonal. The alternating
warp and weft colors then turn that diagonal into a color-and-weave pattern.

---

## Notes for implementers

### Don't trust CONTENTS

Some files list sections that are not in the file, omit sections that are, or write the
manifest malformed. Build your section list by scanning for `[SECTION]` headers. Use
`[CONTENTS]` only as a hint about what to expect. Treat a present-but-empty section as
absent.

### The Date keyline is free text

Real values include `April 20, 1997`, `October 6, 2016`, `Jan 1994`, `10/22/2016`, and
`12/23/95`. There is no format to rely on. Display it as text. If you must turn it into a
date, parse leniently and never reject the file when parsing fails.

### Preinitialize, then apply keylines

See [Default values and sparse data](#default-values-and-sparse-data). Create every end
and pick first, from `[WARP]` / `[WEFT]`, then overlay the keylines that exist. Never
assume a row section lists every index or lists them in order.

### Tie-up and treadling vs lift plan

**Reading.** A draft is defined by *either* a tie-up plus treadling *or* a lift plan. When
a file has both, use the tie-up and treadling; use the lift plan only when the tie-up or
the treadling is missing.

**Writing.** Write the lift plan for every draft — some programs read only the lift plan
and ignore the tie-up entirely. For a floor-loom draft, also write the tie-up and
treadling, so programs and weavers that work in those terms get them. This is what the
current WeaveIt applications do; older versions wrote one or the other.

### Rising vs sinking shed

`Rising Shed=1` raises the shafts named in the tie-up / lift plan; `0` lowers them, which
flips which face of the cloth the drawdown shows. Assume rising if the keyline is missing,
and always write it.

### Multi-treadling

A `[TREADLING]` keyline may name more than one treadle. Store treadling as an array, not a
single number, even if your loom model only supports one treadle per pick.

### Why a file fails to open elsewhere

The usual causes: no `[COLOR PALETTE]` / `[COLOR TABLE]`; a malformed `[CONTENTS]`;
non-UTF-8 bytes; or a private-section line that begins with `[`.

---

## Obsolete and unsupported

The following are part of the historical format but are ignored by current weaving
software, WeaveIt included. Read them only for compatibility; never write them.

### Symbols

`[WARP SYMBOL PALETTE]`, `[WEFT SYMBOL PALETTE]`, `[WARP SYMBOL TABLE]`,
`[WEFT SYMBOL TABLE]`, `[WARP SYMBOLS]`, `[WEFT SYMBOLS]`.

These let a draft be drawn with a letter or mark per yarn instead of a color. Modern
programs, including WeaveIt, do not use them. If you must read them:

- `[* SYMBOL PALETTE]` has `Entries=n`, the size of the symbol table.
- `[* SYMBOL TABLE]` maps `index` → a **WIF Symbol** value.
- `[* SYMBOLS]` maps `end` (or `pick`) → a symbol table index.

A **WIF Symbol** value is one of:

| Form | Example | Meaning |
| --- | --- | --- |
| decimal code | `42` | character code 42 (`*`) |
| literal character | `X` | the character `X` |
| quoted character | `'X'` | the character `X` |
| `#` + code | `#219` | character code 219 |

A space must be written `' '` or `#32`; a `#` must be written `'#'` or `#35`.

### Zoom

`[WARP SPACING ZOOM]`, `[WARP THICKNESS ZOOM]`, `[WEFT SPACING ZOOM]`,
`[WEFT THICKNESS ZOOM]`, and the `Spacing Zoom` / `Thickness Zoom` keylines in `[WARP]` /
`[WEFT]`. A magnification factor for variable-spacing drafts. Not implemented in current
software.

### Bitmap

`[BITMAP FILE]`, `[BITMAP IMAGE]`, `[BITMAP IMAGE DATA]`. Reserved in 1996, never
implemented; support was suspended at version 1.1.

### Translations

`[TRANSLATIONS]`. Intended for localizing key names. Suspended at version 1.1.

### Obsolete keylines

Silently ignore these; they are left in the format only for completeness:

- `[WIF]`: `Developers=` and any email keyline. Deprecated here on privacy grounds — a
  shared file should not carry a contact address. Ignore on read; never write it.
- `[TEXT]`: `Address=`, `EMail=`, `Telephone=`, `FAX=` — same reason.
- `[WARP]` / `[WEFT]`: `Colors=`, `Palette=`, `ColorMix=` (superseded by the color table).
- `[WEAVING]`: `Profile=`.
- The `[DESIGN]` section (dropped at version 1.1).

---

## Sharing `.wif` files on macOS and iOS

Apple platforms identify file types by Uniform Type Identifier (UTI), not by extension, so
every app that reads or writes WIF should declare the same shared UTI,
`org.wif-format.wif`. See **[Sharing the `.wif` File Type on macOS and iOS](apple-uti-setup.md)**
for the `Info.plist` setup in Xcode, including a transitional setup for apps that already
shipped with a private WIF UTI.

---

## Proposals

The format has not changed since 1997. Two draft proposals for how it might evolve are open for
discussion — nothing in them is decided, and neither is part of this guide yet:

- **[WIF 1.2](https://github.com/WIF-Format/WIFStandard/pull/2)** — a conservative revision that
  stays in the INI text format: standardize the extensions that already ship in the field, and
  formally retire the parts nothing implements.
- **[WIF 2](https://github.com/WIF-Format/WIFStandard/pull/1)** — a larger rewrite to JSON with a
  published schema, normalizing the data model.

Comments welcome on either pull request.

---

## License and attribution

**The WIF format** is the work of Ravi Nielsen (Maple Hill Software) and the 1996 and 1997
WIF developer groups — see [Credits](#credits). Nothing in this document claims any right
in the format itself, and no one owns it; it is a shared standard.

**This document** is an independent, plain-language description of that format. It does not
copy or adapt the original specification. The original WIF 1.1 text,
[Copyright © 1996, 1997 Ravi Nielsen](http://www.tantradharma.com/maplehill/wif/wif1-1.txt),
remains the historical source and is not reproduced here.

Copyright © 2026 Shannon Young. This document is licensed under the
[Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)
(CC BY 4.0). You are free to copy, translate, adapt, and redistribute it, for any purpose
including commercially, as long as you give appropriate credit and indicate any changes.
See [`LICENSE`](LICENSE).
