# Weaving Information File (WIF) 2 — Guide

> **Draft proposal, not a standard.** Nothing here is decided, and no software should implement
> it yet. The reasoning and open questions are in the [proposal](proposal.md).

WIF 2 is a proposed successor to the 1997 WIF format, written in
[JSON](https://www.json.org/) instead of INI text. It describes the same thing — a weaving
draft — but stores it the way a weaver thinks about it: a set of yarns, a warp and a weft
built from those yarns, and the loom they are woven on.

This guide is written for two readers at once: a **weaver**, who knows what a threading or a
tie-up is but not how it is stored, and a **developer**, who needs to read or write the file
correctly. Each section first says what the data means at the loom, then — under **In the
file** — how it is encoded.

### The specification is the schema

WIF 2 has no prose specification. The exact rules — every field, its type, which fields are
required, and which combinations are allowed — are in the JSON Schema
**[`wif2.schema.json`](wif2.schema.json)** (JSON Schema Draft 2020-12). Where this guide and
the schema disagree, the schema wins.

A file is valid WIF 2 when it validates against the schema. Any JSON Schema validator will
check it, for example in Python:

```
python3 -m pip install jsonschema
python3 -c "import json, jsonschema; jsonschema.validate(json.load(open('point-twill.wif2')), json.load(open('wif2.schema.json')))"
```

### What changes from 1.1

- **JSON, not INI.** One JSON object per file, extension `.wif2`.
- **Yarns have names.** Each yarn is listed once, with its color, fiber, and size, under a
  name you choose (`"navy"`). Threads refer to it by that name.
- **Warp and weft are sequences.** Instead of a numbered row per thread, the warp and weft are
  lists of runs — "8 ends of navy on shaft 1" — with repeat blocks.
- **The loom type is explicit.** A floor loom, a lift-plan loom, or a rigid-heddle loom, each
  described in its own terms.
- **Dates are ISO**, notes are one block of text, and contact details are not stored at all.
- **Gone:** symbols, zoom, bitmaps, translations, the table of contents, and the color range.

---

## Terminology

| Term | Meaning |
| --- | --- |
| **end** / **pick** | A single warp thread / a single weft thread. |
| **shaft** | A frame that raises or lowers a group of ends. |
| **treadle** | A foot pedal on a floor loom, tied to one or more shafts. |
| **tie-up** | Which shafts each treadle is connected to. |
| **lift plan** | The shafts raised for each pick, given directly. |
| **rigid heddle** | A single frame of holes and slots. Ends in holes move with the heddle; ends in slots stay put. |
| **sett** / **PPI** | Ends per inch (or cm) / picks per inch (or cm). |
| **object** | A JSON `{ … }` with named fields. |
| **array** | A JSON `[ … ]` list. |
| **id** | A short name you give a yarn or color, such as `"navy"` or `"tabby-white"`. |
| **group** | A run of consecutive ends or picks that are all the same. |
| **repeat block** | A list of groups woven several times in a row. |

---

## The file

A WIF 2 file is a single JSON object. It is **strict JSON**: no comments, no trailing commas.
It is meant to be written and read by programs; a person can read it, but should not have to
hand-edit it.

```json
{
  "$schema": "https://wif-format.org/schema/wif2.schema.json",
  "wif": 2,
  "title": "Point twill sample",
  "author": "Sample",
  "created": "2026-08-30",
  "source": { "program": "WIF 2 example", "version": "0.1" },
  "notes": "First line.\nSecond line.",
  "weaveType": "balanced",
  "loom":    { … },
  "palette": { … },
  "yarns":   { … },
  "warp":    { … },
  "weft":    { … },
  "tieup":   [ … ],
  "extensions": { … }
}
```

**In the file**

| Field | Meaning |
| --- | --- |
| `wif` | Required. Always `2`. |
| `$schema` | Optional. The schema's URL, so editors can check the file. |
| `title`, `author` | Optional. The draft's name, and its author's name — never contact details. |
| `created` | Optional. The date, as `YYYY-MM-DD`. |
| `source` | Optional. The `program` and `version` that wrote the file. |
| `notes` | Optional. Free text; line breaks are written `\n`. |
| `weaveType` | Optional. A hint for shading the drawdown: `balanced`, `warp-faced`, `weft-faced`, `double-weave`, `profile`, `double-width`, or `other`. |
| `importedFrom` | Optional. Set when the file was converted from an older format, e.g. `"wif/1.1"`. |

**Unknown fields.** A reader ignores any field it does not recognize. That is how future
versions add things without breaking today's programs.

---

## The loom

What kind of loom the draft is for, and how many shafts it has. The loom type decides how the
weft says which threads are raised.

**In the file** — the `loom` object.

| Field | Meaning |
| --- | --- |
| `type` | `"shaft"` (default), `"lift-plan"`, or `"rigid-heddle"`. |
| `shafts` | Number of shafts, 1–64. Required for `"shaft"` and `"lift-plan"`. |
| `treadles` | `"shaft"` only. Number of treadles; defaults to the shaft count. |
| `shed` | `"rising"` (default) raises the chosen shafts; `"sinking"` lowers them, which flips the drawdown. |
| `heddles` | `"rigid-heddle"` only. 1 (default), or 2–3 for pickup and pattern work. |

```json
"loom": { "type": "shaft", "shafts": 4, "treadles": 4, "shed": "rising" }
```

| Loom type | For | Warp says | Weft says | Tie-up |
| --- | --- | --- | --- | --- |
| `shaft` | Floor looms | `shaft` | `treadles` | Required |
| `lift-plan` | Table looms, dobbies, software | `shaft` | `shafts` | None |
| `rigid-heddle` | Rigid-heddle looms | `heddle`, `slot` | `shed` | None |

Unlike WIF 1.1, a draft is written one way only. A floor-loom draft has a tie-up and
treadling and no lift plan; a reader that wants a lift plan works it out from the two.

---

## Yarns and colors

Every yarn in the draft is listed once, with a short name. The warp and the weft then say
which yarn each thread is by using that name. Change a yarn here, and every thread made from
it changes too.

**In the file** — `yarns` is an object whose keys are yarn ids. Use readable ids like
`"navy"`, not random codes.

```json
"yarns": {
  "navy": {
    "fiber": "cotton",
    "size": { "label": "20/2", "count": 20, "ply": 2, "system": "cotton-count" },
    "color": "navy",
    "note": "Brand X, dye lot 42"
  }
}
```

| Field | Meaning |
| --- | --- |
| `fiber` | Optional. Free text: `"cotton"`, `"wool"`, `"cotton/linen"`. |
| `size` | Optional. See below. |
| `color` | Optional. Either a palette id (`"navy"`) or a color written in place. |
| `note` | Optional. Brand, colorway, dye lot — anything. |

**Size.** `size.label` is the size the way a weaver writes it — `"20/2"`, `"8/2 Tencel"`,
`"Nm 34/2"` — and it is the only required part. A program can always show it; none has to
understand it. If a program does understand yarn counts, it can also fill in `count`, `ply`,
`system` (`"cotton-count"`, `"worsted"`, `"metric-Nm"`, `"tex"`, …), and `wrapsPerInch`.

**Colors.** A color is `{ "rgb": [r, g, b], "name": "Navy" }`, each channel 0–255; `name` is
optional. Colors can be written straight into a yarn, or kept in the `palette` and referred to
by id — useful when several yarns share a color, or to recolor a whole draft by editing the
palette:

```json
"palette": {
  "navy":    { "name": "Navy",    "rgb": [30, 40, 90] },
  "natural": { "name": "Natural", "rgb": [242, 236, 220] }
}
```

---

## The warp and the weft

The warp is read from one edge to the other; the weft is read pick by pick. Each is a list of
**groups** — runs of threads that are all the same yarn and all threaded or woven the same
way — and **repeat blocks**.

**In the file**

| Field | `warp` | `weft` |
| --- | --- | --- |
| Overall spacing | `sett` — ends per `unit` | `ppi` — picks per `unit` |
| `unit` | `"inch"` (default) or `"cm"` | `"inch"` (default) or `"cm"` |
| `sequence` | Required. The groups. | Required. The groups. |

### Groups

Every group names a `yarn` and how many threads it covers (`count`, default 1). It may
override the overall spacing for just those threads with `spacing` — the distance from one
thread to the next, in the same `unit`.

A **warp group** says where its ends are threaded — exactly one of:

| Field | Loom | Meaning |
| --- | --- | --- |
| `shaft` | shaft, lift-plan | The shaft. `0` means not threaded. |
| `shafts` | shaft, lift-plan | Two or more shafts, for the rare end threaded through more than one. |
| `heddle` | rigid-heddle | Which heddle. Add `"slot": true` for a slot; otherwise it is a hole. |

A **weft group** says which shed it is woven in — exactly one of:

| Field | Loom | Meaning |
| --- | --- | --- |
| `treadles` | shaft | The treadles pressed, e.g. `[1]`, or `[1, 3]` for two at once. `[]` is a blank pick. |
| `shafts` | lift-plan | The shafts raised, e.g. `[1, 2, 3]`. |
| `shed` | rigid-heddle | `"up"`, `"down"`, or `"neutral"`; or one per heddle, e.g. `["up", "neutral"]`. |

### Repeat blocks

A repeat block is `{ "repeat": n, "of": [ … ] }` — the list inside is woven `n` times. Blocks
can contain other blocks, so a threading that is "a point, three times, then a border"
reads exactly that way:

```json
"sequence": [
  { "repeat": 3, "of": [
    { "yarn": "navy", "shaft": 1 },
    { "yarn": "navy", "shaft": 2 },
    { "yarn": "navy", "shaft": 3 },
    { "yarn": "navy", "shaft": 4 },
    { "yarn": "navy", "shaft": 3 },
    { "yarn": "navy", "shaft": 2 }
  ]},
  { "yarn": "natural", "shaft": 1, "count": 4 }
]
```

To get each individual end or pick, a reader **expands** the sequence: write each group out
`count` times, and each block's contents out `repeat` times, in order. The number of ends is
whatever that produces — there is no separate thread count to keep in step.

---

## The tie-up

On a floor loom, which shafts each treadle is tied to.

**In the file** — `tieup` is an array with one entry per treadle, in order. Each entry is the
list of shafts that treadle moves; `[]` is an unused treadle. It appears only when
`loom.type` is `"shaft"`.

```json
"tieup": [
  [1, 2],
  [2, 3],
  [3, 4],
  [1, 4]
]
```

Here treadle 1 moves shafts 1 and 2, treadle 2 moves shafts 2 and 3, and so on.

---

## Extensions

Room for a program to keep data that WIF 2 does not describe — planning figures such as loom
waste and shrinkage, for example — without breaking other programs. This replaces 1.1's
private sections.

**In the file** — `extensions` is an object keyed by a short program id. What is inside is up
to that program; the schema does not check it, and other programs ignore it.

```json
"extensions": {
  "weaveit": { "shrinkage": 0.08, "loomWaste": 24 }
}
```

---

## Worked examples

Three complete files accompany this guide:

| File | Loom | Shows |
| --- | --- | --- |
| [`point-twill.wif2`](point-twill.wif2) | 4-shaft floor loom | Palette, yarn sizes, nested repeats, tie-up and treadling, an extension. |
| [`lift-plan.wif2`](lift-plan.wif2) | 8-shaft dobby | A lift plan with no tie-up; a color written directly in a yarn. |
| [`rigid-heddle-plaid.wif2`](rigid-heddle-plaid.wif2) | Rigid heddle | Holes and slots, heddle up and down, a plaid built from repeats. |

**Reading `point-twill.wif2`.** The loom is a 4-shaft floor loom. There are two yarns, both
20/2 cotton, one navy and one natural. The warp begins with two navy ends on shaft 1 and two
on shaft 2, then repeats a six-end point (`1, 2, 3, 4, 3, 2`, alternating navy and natural)
eight times, and ends with two navy ends each on shafts 3 and 4 — 56 ends at 24 per inch. The
tie-up is a 2-2 twill. The weft presses treadles 1, 2, 3, 4 in turn, alternating navy and
natural, fourteen times — 56 picks. The navy-and-natural alternation in both directions turns
the twill into a color-and-weave pattern.

---

## Notes for implementers

### Validate against the schema

Validate files you write. Validate files you read, too, but be forgiving: report a problem
and open what you can, rather than refusing the whole draft.

### Ignore what you do not recognize

Unknown fields, unknown extension keys, and unknown values in `extensions` are all ignored.
Never drop a file because it has a field you did not expect.

### Expand before you draw

Groups and repeat blocks are a compact way to write the draft, not a separate model. Expand
the warp and the weft into individual ends and picks before computing a drawdown, then
compress again when writing — any compression that expands to the same threads is valid.

### Floor loom to lift plan

For a `"shaft"` loom, the shafts raised on a pick are all the shafts tied to every treadle
listed in that pick's `treadles`. With `"shed": "sinking"`, those shafts go down instead.

### Reading WIF 1.1 files

A converter maps each 1.1 section onto the WIF 2 fields and sets `"importedFrom": "wif/1.1"`.
The section-by-section mapping is in the [proposal](proposal.md#importing-a-11-file).

---

## License

Copyright © 2026 Shannon Young. Licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), like the rest of this repository.
Nothing here claims any right in the WIF format itself; see
[License and attribution](../README.md#license-and-attribution).
