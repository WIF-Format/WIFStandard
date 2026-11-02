# Weaving Information File (WIF) 1.2 — Guide

> **Draft proposal, not a standard.** Nothing here is decided, and no software should implement
> it yet. The reasoning behind each change, and the open questions, are in the
> [proposal](proposal.md). The exact rules are in the [specification](wif-format_v1_2.md).

WIF 1.2 is the WIF format you already know, with a few additions. It is still a plain-text
[INI file](https://en.wikipedia.org/wiki/INI_file), still readable by a person, and every 1.1
file is still a valid 1.2 file. It writes down the extensions that weaving programs already
pass back and forth, and it formally deprecates the parts of the 1997 format that are no longer in use.

This guide is written for two readers at once: a **weaver**, who knows what a threading or a
tie-up is but not how it is stored, and a **developer**, who needs to read or write the bytes
correctly. Each section first says what the data means at the loom, then — under **In the
file** — exactly how it is encoded. Anything new in 1.2 is marked **New in 1.2**.

### What changes from 1.1

- **Yarns.** A new `[YARNS]` section gives each color a name, a fiber, and a size.
- **Repeats.** New `[WARP REPEATS]` / `[WEFT REPEATS]` sections record how a threading or
  treadling is built from repeated blocks.
- **The loom.** `[WEAVING]` gains `Loom Type` (floor loom, lift-plan loom, or rigid heddle),
  `Heddles`, and `Weave Type`.
- **Text and dates.** Files are UTF-8, and dates are written `YYYY-MM-DD`.
- **Contents.** `[CONTENTS]` becomes a hint. Readers find sections by looking for them.
- **Deprecated.** Symbols, zoom, bitmaps, translations, and personal contact details are
  formally marked as deprecated.

### Compatibility

A 1.1 program opens a 1.2 file and simply skips what it does not recognize; the draft still
weaves the same. A 1.2 program opens a 1.1 file with no changes. Nothing in 1.1 changes
meaning — every new thing has a new name.

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
| **treadle** | A foot pedal on a floor loom. Pressing it moves the shafts tied to it. |
| **threading** | Which shaft each end passes through, read across the warp. |
| **tie-up** | Which shafts each treadle is connected to. |
| **treadling** | The order in which the treadles are pressed, one entry per pick. |
| **lift plan** | The shafts raised for each pick, given directly — used instead of a tie-up and treadling. |
| **rising / sinking shed** | Whether the loom raises the selected shafts (jack loom) or lowers them (countermarch, counterbalance). |
| **rigid heddle** | A single frame of alternating holes and slots. Ends in holes move when the heddle is raised or lowered; ends in slots stay put. |
| **yarn** | A physical thread: a color, a fiber, and a size such as `10/2`. |
| **repeat** | A block of threading or treadling that is woven several times in a row. |
| **sett** | How closely the ends are spaced, usually in ends per inch (EPI). Picks per inch (PPI) is the weft equivalent. |
| **drawdown** | The picture of the finished cloth produced by combining threading, tie-up / lift plan, and treadling. |

### In the file

| Term | Meaning |
| --- | --- |
| **section** | A named block, written `[SECTION NAME]`, containing keylines. |
| **keyline** | One `Key=Value` line inside a section. |
| **map** | A section whose keys are either integers (`1`, `2`, `3`, …) or names. |
| **array** | A comma-separated list of values in a single keyline, e.g. `1,3,5`. |

---

## Data types

| Type | Notes |
| --- | --- |
| **Integer** | A whole number. Treat it as 32-bit. |
| **Real** | A decimal number with a period, e.g. `0.05` or `24`. |
| **String** | Text, in UTF-8. |
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
  follow a number or boolean value to comment it. After a string value a `;` is literal text.
- **Case.** Section and key names are case-insensitive.
- **Order.** Sections, and keylines within a section, may appear in any order.
- **Duplicate keys.** Writing the same key twice in one section is an error. A reader keeps
  one and does not crash.
- **Line endings.** CRLF or LF, and sometimes both in one file. Accept any.
- **Finding sections.** A reader finds the sections by scanning for `[SECTION]` headers, not by
  trusting `[CONTENTS]`. A section with no keylines counts as absent.

### Default values and sparse data

The row sections — `[THREADING]`, `[TREADLING]`, `[WARP COLORS]`, `[WARP SPACING]`, and so on
— do **not** have to list every end or pick. A reader:

1. Creates `[WARP].Threads` ends and `[WEFT].Threads` picks up front.
2. Gives each one the default `Color`, `Spacing`, and `Thickness` from `[WARP]` / `[WEFT]`,
   and sets every shaft and treadle to `0`, meaning *unused*.
3. Then applies whatever keylines the file contains, overriding those defaults.

So a `[WARP COLORS]` section that contains only `3=2` means *every end is the default warp
color except end 3, which is color 2*. A writer may leave out any keyline that equals the
default, or write them all.

### Text encoding — New in 1.2

A 1.2 file is **UTF-8**. It may say so with `[WIF] Encoding=UTF-8`. Older files are often
ISO-8859-1 (Latin-1), so a reader tries UTF-8 first and falls back to Latin-1.

---

## Header sections

### WIF

Identifies the file as a WIF and records which program wrote it.

**In the file** — required.

```
[WIF]
Version=1.2                 ;required
Source Program=Mac WeaveIt  ;required. The program that wrote the file.
Source Version=1.0          ;optional. That program's version.
Date=2026-10-06             ;required
Encoding=UTF-8              ;optional. New in 1.2.
```

- **`Version`** is `1.2`. A program that uses none of the 1.2 additions may still write `1.1`.
  Readers display the version and never refuse a file over it.
- **`Date`** — **New in 1.2:** write it as `YYYY-MM-DD`. Older files contain anything at all
  (`April 20, 1997`, `10/22/2016`, `Jan 1994`), so readers should still show the value as
  text when it does not parse, and never reject the file.

_Do not write a `Developers` or email keyline. A WIF file travels through forums and email
attachments, and an address in it is a privacy and spam risk._

### CONTENTS

A table of contents listing the other sections.

**In the file** — optional. One `NAME=1` keyline per section present.

```
[CONTENTS]
WEAVING=1
WARP=1
WEFT=1
THREADING=1
LIFTPLAN=1
```

**Changed in 1.2:** `[CONTENTS]` is a hint only. Readers find sections by scanning the file.
Writers may still include it for older programs; if they do, it must be correct.

---

## Project sections

### TEXT

The title and author of the draft.

**In the file** — optional.

```
[TEXT]
Title=Point twill, color & weave
Author=Sally Breckenridge
```

_Never write `Address`, `EMail`, `Telephone`, or `FAX` — they are personal contact details
that do not belong in a file meant to be shared. Ignore them on read._

### NOTES

Free-text notes from the person who made the draft.

**In the file** — optional. One keyline per line of text, numbered from 1. A reader joins the
lines with newlines.

```
[NOTES]
1=This is a 2-2 twill double cloth. It is woven top-bottom-bottom-top.
2=Change the colors to create your own twill double cloth.
```

---

## The loom

### WEAVING

Describes the loom: how many shafts and treadles it has, whether it raises or lowers the
shafts, and — **new in 1.2** — what kind of loom it is and how the cloth is meant to look.

**In the file** — required whenever the file has a threading, tie-up, treadling, or lift plan.

```
[WEAVING]
Shafts=8              ;required (Integer > 0)
Treadles=8            ;required (Integer > 0). Use the shaft count if there are no treadles.
Rising Shed=1         ;always write it
Loom Type=Shaft       ;optional. New in 1.2. Shaft | Lift Plan | Rigid Heddle
Weave Type=Balanced   ;optional. New in 1.2.
```

**`Rising Shed`.** `1` means the loom raises the shafts named in the tie-up or lift plan (a
jack loom); `0` means it lowers them (countermarch or counterbalance), which flips the
drawdown. A 1.2 file always writes it. If it is missing, assume `1`.

**`Loom Type`** — New in 1.2. Tells a reader what kind of loom the draft is for, and so which
part of the draft to trust. It never changes the threading or the drawdown, so a 1.1 reader
that ignores it still draws the right cloth.

| Value | Loom | What it means for the draft |
| --- | --- | --- |
| `Shaft` (or absent) | A floor loom with treadles. | The tie-up and treadling are the draft. A lift plan, if present, is a copy for programs that want one. |
| `Lift Plan` | A table loom, dobby, or software. | The lift plan is the draft. Any tie-up and treadling are for reference. |
| `Rigid Heddle` | A rigid-heddle loom. | See below. |

**Rigid heddle.** A rigid-heddle draft is written as an ordinary 2-shaft draft, which is what
rigid-heddle WIF files already look like:

- shaft 1 is the **holes** of the heddle, shaft 2 is the **slots**;
- plain weave is the threading `1,2,1,2,…` with picks that alternate raising shaft 1 and
  raising shaft 2 — heddle up, heddle down.

`Loom Type=Rigid Heddle` simply lets a 1.2 program show holes, slots, and heddle up/down in
place of shafts. For pickup or pattern work with two heddles, use 4 shafts (shafts 3 and 4
are the second heddle's holes and slots) and add `Heddles=2`.

```
[WEAVING]
Shafts=2
Treadles=2
Rising Shed=1
Loom Type=Rigid Heddle
Heddles=1              ;optional. 1 (default), 2, or 3.
```

**`Weave Type`** — New in 1.2. A hint about how to shade the drawdown: `Balanced`,
`Warp-faced`, `Weft-faced`, `Double-weave`, `Profile`, or `Double-width`. Readers also accept
the older tokens `BALANCE`, `WARPFACE`, `WEFTFACE`, `DOUBLEWEAVE`, `PROFILEDRAFT`, and
`DOUBLEWIDTH`.

---

## The warp and the weft

### WARP and WEFT

The setup for the warp and the weft as a whole: how many ends or picks there are, and their
default color, spacing, and thickness. Individual ends and picks can override these.

**In the file** — both sections have the same keylines.

```
[WARP]         (and, identically, [WEFT])
Threads=8      ;required (Integer > 0). The number of ends (or picks).
Color=1        ;optional. Default color, as an index into the COLOR TABLE.
Units=Inches   ;required if Spacing or Thickness is present. Inches | Centimeters.
Spacing=0.05   ;optional (Real >= 0). Distance between ends, in Units — 1 / sett.
Thickness=0.05 ;optional (Real >= 0). Drawn width of the yarn, in Units.
```

- Older files may write `Color` as `index,R,G,B`. Read the first number.
- Older files may write `Units=Decipoints`. Treat it as `Inches`; never write it.

---

## Threading and drawdown

A draft gives the threading plus **either** a tie-up and treadling **or** a lift plan. It may
carry both; `Loom Type` says which one is the real draft.

### THREADING

Which shaft each warp end passes through, read across the warp from one edge.

**In the file** — a map from end number to shaft number.

```
[THREADING]
1=1     ;end 1 on shaft 1
2=2
3=3
4=4
```

- `1=1,5` means end 1 passes through heddles on both shaft 1 and shaft 5. This is rare; a
  reader that cannot handle it uses the first shaft.
- An end with no keyline is on shaft `0` — not threaded.

### TIEUP

On a floor loom, which shafts each treadle is tied to.

**In the file** — a map from treadle number to the shafts it moves.

```
[TIEUP]
1=1,2   ;treadle 1 moves shafts 1 and 2
2=2,3
3=3,4
4=1,4
```

A treadle with the value `0`, or with no keyline, is unused.

### TREADLING

The order the treadles are pressed, one entry per pick.

**In the file** — a map from pick number to the treadles pressed.

```
[TREADLING]
1=1
2=2
3=3
4=4
```

More than one treadle (`5=1,3`) means they are pressed together. `0` is a pick with no treadle.

### LIFTPLAN

The shafts raised for each pick, given directly.

**In the file** — a map from pick number to the shafts raised.

```
[LIFTPLAN]
1=1,2
2=2,3
3=3,4
4=1,4
```

**Which to use.** With `Loom Type=Lift Plan`, use the lift plan. Otherwise use the tie-up and
treadling, and fall back to the lift plan only if either is missing.

**What to write.** Write a lift plan for every draft, because some programs read nothing else.
For a floor-loom draft, also write the tie-up and treadling.

---

## Color and yarn

### COLOR PALETTE and COLOR TABLE

The set of colors used in the draft.

**In the file**

```
[COLOR PALETTE]
Entries=2     ;required. Number of rows in COLOR TABLE.
Range=0,255   ;required. Min and max for each R, G, B channel.

[COLOR TABLE]
1=32,32,32    ;color 1 as R,G,B
2=240,240,240
```

- `Range` is almost always `0,255`. If not, scale each channel with
  `(value - min) * 255 / (max - min)`.
- **Include these sections** whenever the draft has color. Many programs refuse a file with
  colored threads and no color table.

### YARNS — New in 1.2

The color table says only what color a thread is. `[YARNS]` adds what it *is*: a name, a
fiber, and a size. It comes from the private yarn section the WeaveIt applications already
write.

**In the file** — one row per `[COLOR TABLE]` row, with the same numbers.

```
[YARNS]
1=246,238,196, Yellow, cotton, 10/2
2=196,235,238, Blue, cotton, 5/2
```

- Each row is `R, G, B, name, fiber, size`. The last three are free text and may be left
  empty; existing files sometimes write `NoValue` instead.
- Always write `[COLOR TABLE]` too, with the same colors, so 1.1 programs still see them.
- A reader that finds `[YARNS]` uses it; otherwise it uses `[COLOR TABLE]` alone.

### WARP COLORS and WEFT COLORS

The color of each end or pick, as a color-table number. With the threading, this is what
produces stripes, plaids, and color-and-weave effects.

**In the file**

```
[WARP COLORS]        (and, identically, [WEFT COLORS])
2=2     ;end 2 uses color 2; all other ends use [WARP].Color
4=2
```

An end or pick with no keyline uses the default `Color` from `[WARP]` / `[WEFT]`.

---

## Spacing and thickness

### WARP SPACING, WARP THICKNESS, WEFT SPACING, WEFT THICKNESS

Per-end and per-pick overrides of `Spacing` and `Thickness` — for drafts that mix thick and
thin yarns or change the sett partway across.

**In the file** — a map from end (or pick) number to a value in the `[WARP]` / `[WEFT]` `Units`.

```
[WARP SPACING]       (and, identically, [WEFT SPACING])
5=0.10
6=0.10

[WARP THICKNESS]     (and, identically, [WEFT THICKNESS])
5=0.08
6=0.08
```

---

## Repeats — New in 1.2

### WARP REPEATS and WEFT REPEATS

Weavers think of a threading as blocks — "this 7-end point, three times." The threading
section stores every end individually, so that structure is lost. These sections keep it, so
a program can show and edit the blocks.

**In the file** — each row is `location, count, length`: starting at end (or pick)
`location`, a block of `length` threads is repeated `count` times.

```
[WARP REPEATS]
1=0,3,7        ;from the first end, a 7-end block three times
2=21,2,14
```

- `location` counts from **0**, unlike every other number in WIF.
- These sections are **only a hint**. The full threading and treadling are still written out
  in `[THREADING]`, `[TREADLING]`, and `[LIFTPLAN]`, and a reader may ignore the repeats.

---

## Private sections

Room for a program to store data that WIF does not cover, without breaking other programs.

**In the file** — the section name is `[PRIVATE <SourceID> <SectionName>]`, where
`<SourceID>` is a short tag for the program. Other programs skip it. **No line inside a
private section may begin with `[`.**

```
[PRIVATE WEAVEIT AWESOME]
Sparkles=1
```

---

## A complete example

A 2-2 twill on 4 shafts, alternating dark and light in both warp and weft, written as a 1.2
floor-loom draft. It carries a lift plan for programs that want one, and a yarn table. The
same file is [`twill-color-and-weave.wif`](twill-color-and-weave.wif).

```
[WIF]
Version=1.2
Source Program=Example
Date=2026-10-06
Encoding=UTF-8

[COLOR PALETTE]
Entries=2
Range=0,255

[COLOR TABLE]
1=32,32,32
2=240,240,240

[YARNS]
1=32,32,32, Charcoal, cotton, 10/2
2=240,240,240, Natural, cotton, 10/2

[WEAVING]
Shafts=4
Treadles=4
Rising Shed=1
Loom Type=Shaft
Weave Type=Balanced

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

[WARP REPEATS]
1=0,2,4

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

A 1.1 program reads this as the same twill it has always read: it skips `[YARNS]`,
`[WARP REPEATS]`, `Loom Type`, `Weave Type`, and `Encoding`, and finds everything else where
it expects it. There is no `[CONTENTS]`; a 1.2 writer that wants to support very strict 1.1
readers can add one.

---

## Deprecated

These are part of the historical format. A 1.2 reader ignores them; a 1.2 writer never
writes them.

- **Symbols** — `[WARP SYMBOL PALETTE]`, `[WEFT SYMBOL PALETTE]`, `[WARP SYMBOL TABLE]`,
  `[WEFT SYMBOL TABLE]`, `[WARP SYMBOLS]`, `[WEFT SYMBOLS]`, and the `Symbol` / `Symbol Number`
  keylines in `[WARP]` / `[WEFT]`. Drawing yarns as letters instead of colors.
- **Zoom** — `[WARP SPACING ZOOM]`, `[WARP THICKNESS ZOOM]`, `[WEFT SPACING ZOOM]`,
  `[WEFT THICKNESS ZOOM]`, and the `Spacing Zoom` / `Thickness Zoom` keylines. Never built.
- **Bitmap** — `[BITMAP FILE]`, `[BITMAP IMAGE]`, `[BITMAP IMAGE DATA]`. Never built.
- **`[TRANSLATIONS]`** and **`[DESIGN]`** — dropped in 1.1.
- **Contact details** — `[WIF] Developers`, and `[TEXT] Address`, `EMail`, `Telephone`, `FAX`.
- **Old color keylines** — `[WARP]` / `[WEFT]` `Colors`, `Palette`, `ColorMix`.
- **`[WEAVING] Profile`**, and **`Units=Decipoints`**.

---

## License

Copyright © 2026 Shannon Young. Licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), like the rest of this repository.
Nothing here claims any right in the WIF format itself; see
[License and attribution](../README.md#license-and-attribution).
