# Weaving Information File (WIF) 1.1 — Guide

WIF 1.1 is the current WIF format, unchanged since 1997. It is a plain-text format, based
on the [INI file](https://en.wikipedia.org/wiki/INI_file) convention, and it was designed
to be read by a person as easily as by a program. For the format's history, see the
[overview](../README.md); ideas for future versions are on
[Proposals under discussion](../proposals.md).

This guide is a plain-language description of WIF 1.1 as it is actually used today,
written for two readers at once: a **weaver**, who knows what a threading or a tie-up is
but not how it is stored, and a **developer**, who needs to read or write the bytes
correctly. Each section below first says what the data means at the loom, then — under
**In the file** — exactly how it is encoded.

The same format is also written up as reference tables in the
[WIF 1.1 specification](wif-format_v1_1.md), and the example below is
[`twill-color-and-weave.wif`](twill-color-and-weave.wif).

> **Building an app for Mac, iPhone or iPad?** Before you ship, follow the
> [Developer Guide: Sharing the .wif Format on macOS and iOS](../ios-macos-uti/README.md). An app
> that declares the `.wif` file type the wrong way can stop every other weaving app on the
> same device from opening `.wif` files.

---

## Terminology

For weaving words such as *warp*, *end*, *shaft* and *tie-up*, see
[Weaving terms](../glossary.md). This guide also uses these words for the parts of the file:

| Term | Meaning |
| --- | --- |
| **section** | A named block, written `[SECTION NAME]`, containing keylines. |
| **keyline** | One `Key=Value` line inside a section. |
| **map** | A section whose keys are either integers (`1`, `2`, `3`, …) or names. WIF sections are all maps. |
| **array** | A comma-separated list of values in a single keyline, for example `1,3,5`. |

---

## Data types

| Type | Notes |
| --- | --- |
| **Integer** | A whole number. Treat it as 32-bit. |
| **Real** | A decimal number, with a period as the decimal point, for example `0.05` or `24`. The 1997 text calls this *real*; some call it *double*. |
| **String** | Text. The 1997 text says ASCII; in practice see [Text encoding](#text-encoding). |
| **Boolean** | True is any of `true`, `on`, `yes`, `1`; false is any of `false`, `off`, `no`, `0`. An absent optional boolean is false, except `Rising Shed` (see [WEAVING](#weaving)). |
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
- **Empty sections.** A section with no keylines is treated as absent.

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
Developers=Canyon Art       ;optional. The company that makes the program. Never an email.
Date=October 6, 2016        ;required. Free text — see the note below.
```

**`Developers` differs from the 1997 text.** The 1997 text describes `Developers` as a
contact email address and makes it required. In practice, programs write the name of the
company or developer that makes them: TempoWeave writes `Developers=LoftyFiber`, and WeaveIt
for Windows writes `Developers=Canyon Art`. This tells apart programs whose `Source Program`
names are alike. This guide follows that practice:

- `Developers` is **optional**. A reader must not require it.
- When present, it holds a **company or developer name**.
- It is **never an email address**. A WIF file travels through forums and email
  attachments; an address in one is a privacy and spam risk to whoever put it there, and no
  reader needs it. A reader should ignore an email address found there.

`Date` has no dependable format — see
[Notes for implementers](#the-date-keyline-is-free-text).

### CONTENTS

A table of contents listing the other sections in the file.

**In the file** — required by the 1997 text, so always write it. Boolean keylines keyed by
section name; only `true` entries are meaningful. When reading, treat it as a **hint, not the
source of truth**: scan the
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
LIFTPLAN=1
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
Units=Inches   ;required if Spacing, Thickness, or any spacing or thickness section
               ;is present. Inches or Centimeters.
Spacing=0.05   ;optional (Real >= 0). Center-to-center distance between ends, in Units
               ;— that is, 1 / sett. 0.05 in = 20 ends per inch.
Thickness=0.05 ;optional (Real >= 0). Visual width of the yarn, in Units.
```

- `Color` may appear as a bare index (`1`) or, in older files, as `index,R,G,B`. Read the
  first number; ignore any trailing RGB. Decode it as an array so old files still load.

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
  rare; see [Ends on more than one shaft](#ends-on-more-than-one-shaft) for what it means at
  the loom and in a drawdown.
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
Range=0,255   ;required (Array of two Integers). Min and max value for each R, G, B channel.

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
recognize a private section skips it, so the section is lost if that program saves the file
again. **No line inside a private section may begin with `[`** — that is how a reader finds
where the next real section starts.

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
LIFTPLAN=1
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

[LIFTPLAN]
1=1,2            ;the same lifts as the tie-up and treadling, for programs
2=2,3            ;that read only the lift plan
3=3,4
4=1,4
5=1,2
6=2,3
7=3,4
8=1,4

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

### Ends on more than one shaft

A `[THREADING]` keyline may name more than one shaft, such as `1=1,5`. At the loom this
needs *long-eyed heddles*: their tall eye lets the end rise with one shaft without being
held down by the other. With ordinary heddles, the two shafts pull the end in opposite
directions, and the shed does not open cleanly. Weavers use it for double-harness work such
as damask, which has one set of shafts for the pattern and another for the ground.

- **In the drawdown,** on a rising-shed loom, the end is up on a pick if **any** of its
  shafts is raised.
- **Store threading as an array**, as you would treadling.
- **If your program allows only one shaft per end,** keep the first shaft, and tell the
  user that the drawdown may differ from the original. Never treat such an end as
  unthreaded.

### Multi-treadling

A `[TREADLING]` keyline may name more than one treadle. Store treadling as an array, not a
single number, even if your loom model only supports one treadle per pick.

### Why a file fails to open elsewhere

The usual causes: no `[COLOR PALETTE]` / `[COLOR TABLE]`; a malformed `[CONTENTS]`;
non-UTF-8 bytes; or a private-section line that begins with `[`.

---

## Obsolete and unsupported

This guide limits itself to what the [Canyon Art WeaveIt](https://weaveit.com/) applications
read and write. Their author was one of the original WIF 1.1 developers, so where those
applications ignore a part of the format, this guide treats that part as deprecated and
describes it only briefly here.

Each deprecated part follows one of two rules:

| Rule | Applies to | Why |
| --- | --- | --- |
| **[Do not write](#do-not-write)** | Email addresses and other contact details, zoom, bitmap, translations, design, the old color keylines, `Profile`, `Units=Decipoints` | Each one exposes personal information, was never implemented, has been replaced, or means different things to different programs. The reason for each is below. |
| **[Do not depend on other programs keeping it](#do-not-depend-on-other-programs-keeping-it)** | Symbols, and [private sections](#private-sections) | Writing them is allowed. But most programs skip them, and drop them when they save the file again. |

### Do not write

A reader may come across these in older files and can read them for compatibility. A
writer should never write them, for the reason given:

| Part | Why not to write it |
| --- | --- |
| An email address in `[WIF]` `Developers=`, or any other email keyline | **Personal information.** A WIF file travels through forums and email; an address in it is a privacy and spam risk to whoever put it there, and no reader needs it. Ignore it when reading. `Developers` itself may hold a company name; see [WIF](#wif). |
| `[TEXT]`: `Address=`, `EMail=`, `Telephone=`, `FAX=` | **Personal information**, for the same reason. Ignore it when reading. |
| `[WARP SPACING ZOOM]`, `[WARP THICKNESS ZOOM]`, `[WEFT SPACING ZOOM]`, `[WEFT THICKNESS ZOOM]`, and the `Spacing Zoom` / `Thickness Zoom` keylines in `[WARP]` / `[WEFT]` | **Never implemented.** A magnification factor for variable-spacing drafts that no current program uses, so no reader would apply it. |
| `[BITMAP FILE]`, `[BITMAP IMAGE]`, `[BITMAP IMAGE DATA]` | **Never implemented.** Reserved in 1996; support was suspended at version 1.1. |
| `[TRANSLATIONS]` | **Never implemented.** Meant for translating key names; suspended at version 1.1. |
| `[DESIGN]` | **Removed** at version 1.1. |
| `[WARP]` / `[WEFT]`: `Colors=`, `Palette=`, `ColorMix=` | **Replaced** by `[COLOR PALETTE]` and `[COLOR TABLE]`. Writing both risks two different colors for the same thread. |
| `[WEAVING]`: `Profile=` | **Obsolete.** No current program uses it. |
| `[WARP]` / `[WEFT]`: `Units=Decipoints` | **Programs disagree on what it means.** Some read it as inches; others convert it at 720 decipoints to the inch. Write `Inches` or `Centimeters`. When reading, treat it as `Inches`. |

### Do not depend on other programs keeping it

These are still part of WIF 1.1, and writing them is allowed. But most weaving programs,
WeaveIt included, skip them, and a program that skips a section also drops it when it saves
the file again. Write them if your program uses them, but don't put anything there that the
draft can't do without.

- **[Private sections](#private-sections)**, which hold one program's own data.
- **Symbols**, below.

#### Symbols

`[WARP SYMBOL PALETTE]`, `[WEFT SYMBOL PALETTE]`, `[WARP SYMBOL TABLE]`,
`[WEFT SYMBOL TABLE]`, `[WARP SYMBOLS]`, `[WEFT SYMBOLS]`.

These let a draft be drawn with a letter or mark per yarn instead of a color. Most
programs, including the WeaveIt applications, do not use them. If you read or write them:

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

<div class="github-only" markdown="1">

---

© 2026 the WIF-Format contributors. Licensed under [CC BY 4.0](../LICENSE). The WIF format is by Ravi Nielsen (Maple Hill Software) and the 1996–97 WIF developer groups; original text © 1996, 1997 Ravi Nielsen. The format itself is a shared standard, owned by no one.

</div>
