# WIF 2 — a draft proposal

> **This is an early draft for discussion, not a standard.** Nothing here is decided.
> It is not linked from the main WIF format guide and no software should implement it yet.

## What this is

A sketch of a JSON successor to the 1997 WIF text format:

- [`wif2.schema.json`](wif2.schema.json) — a JSON Schema (Draft 2020-12) for the proposed format.
- [`point-twill.wif2`](point-twill.wif2) — a worked example: a 4-shaft floor loom (tie-up + treadling).
- [`lift-plan.wif2`](lift-plan.wif2) — a worked example: an 8-shaft table/dobby loom (lift plan, no tie-up).
- [`rigid-heddle-plaid.wif2`](rigid-heddle-plaid.wif2) — a worked example: a rigid-heddle loom.

Files use the **`.wif2`** extension and are **strict JSON** — no comments. They are written and read
by tools, not hand-authored.

## Why JSON, and why a schema

The 1997 format has served for 30 years and its longevity is the whole point: drafts must outlive the
software that made them. A successor has to be at least as durable, which rules out anything clever.
JSON is old — and for an archival interchange format that is exactly why it fits. Every language,
browser, and microcontroller will still parse it in 2055, and its specification is frozen. Binary
formats (CBOR, Protobuf) buy compression and speed that a few-kilobyte weaving draft does not need;
an API query language like GraphQL is not a file format at all.

What has actually aged about JSON is being *schemaless*. Paired with a published schema and a test
corpus, it gives the modern experience — typed, validated, tool-checked — on a substrate that will
not move. So the real deliverable is the schema, not the syntax.

## What changes from 1.1

The 1997 format stores what is really a small relational model as parallel, position-indexed sections.
A **yarn** — a physical thread with a fibre, a size, and a colour — has no identity in the file, so
its attributes are scattered across `[COLOR TABLE]`, `[WARP COLORS]`, `[WARP SPACING]`, and
`[PRIVATE IWEAVEIT YARNS]`, and every reader has to rejoin them by position.

WIF 2 normalizes this:

- **Yarns and colours are entities** with stable, human-meaningful string ids (`"navy"`, not a UUID),
  in a `yarns` map and an optional `palette` map. Warp and weft groups reference a yarn by id.
- **The warp and weft are ordered `sequence`s of run-length groups** —
  `{ "yarn": "navy", "shaft": 1, "count": 8 }` — with recursive **repeat blocks**
  `{ "repeat": 3, "of": [ … ] }`. This replaces the 1997 sparse-data rules *and* the
  `[PRIVATE WEAVEIT WARP REPEATS]` mechanism with one first-class concept.
- **The loom mechanism is explicit.** `loom.type` is `"shaft"` (floor loom, treadles + a `tieup`),
  `"lift-plan"` (table loom, dobby, drawloom, or software — weft groups list the raised `shafts`
  directly, no `tieup`), or `"rigid-heddle"` (modelled directly with `heddle`/`slot` and
  `shed: up/down/neutral`, not shoehorned into a 2-shaft loom).
- **`created` is an ISO date**, not free text. **`notes` is a single string**, newlines allowed.
  Contact information (email, phone, address) is not modelled at all.
- **Vendor data goes in `extensions`** — a bag keyed by vendor id that the schema does not describe.
  Project-planning figures (loom waste, shrinkage, finished dimensions) live here, the way private
  sections worked in 1.1.
- **`wif: 2`** marks the version. Unknown properties are allowed everywhere and reserved for future
  minor versions — a reader must ignore what it does not recognise.

Everything the WIF format guide already lists as obsolete — symbols, zoom, bitmap, translations —
is simply gone.

## Importing a 1.1 file

A converter reads a 1.1 file and writes v2 with `"importedFrom": "wif/1.1"`.

| WIF 1.1 | WIF 2 |
|---|---|
| `[WIF]` `Source Program` / `Source Version` | `source.program` / `source.version` |
| `[WIF]` `Date` (free text) | `created` — parse leniently to an ISO date; drop it if unparseable |
| `[TEXT]` `Title` / `Author` | `title` / `author` |
| `[NOTES]` lines `1..n` | joined with `\n` into the `notes` string |
| `[WEAVING]` `Shafts` / `Treadles` / `Rising Shed` | `loom` — `type: "shaft"`, `shafts` / `treadles` / `shed` (`1` → `"rising"`, `0` → `"sinking"`) |
| `[COLOR PALETTE].Range` + `[COLOR TABLE]` | `palette` — rescale each channel to 0–255 with `(v - min) * 255 / (max - min)` |
| `[PRIVATE IWEAVEIT YARNS]` rows `R,G,B,name,yarnType,yarnWidth,variableSett` | `yarns[id]` — `color` (+ `color.name` ← name), `size.label` ← yarnType, `fiber` ← the fibre words in yarnType; `variableSett` → a per-group `spacing` of `1 / variableSett` |
| `[WARP]` / `[WEFT]` `Threads` / `Color` / `Units` / `Spacing` | `warp` / `weft` — `sett = round(1 / Spacing)`, `unit` (`Decipoints` → `"inch"`), and the default yarn colour |
| `[THREADING]` + `[WARP COLORS]` + `[PRIVATE WEAVEIT WARP REPEATS]` | `warp.sequence` — pair shaft + yarn per end, run-length compress; use the repeat rows (`location, repeatCount, length`) to build `{ repeat, of }` blocks |
| `[TREADLING]` + `[TIEUP]` | `loom.type: "shaft"`, `tieup` array, `weft.sequence` groups with `treadles` |
| `[LIFTPLAN]` (no tie-up / treadling) | `loom.type: "lift-plan"`, `weft.sequence` groups with `shafts`, no `tieup` |
| a file carrying both (the 1.1 belt-and-suspenders convention) | prefer the tie-up + treadling → `loom.type: "shaft"` |
| `[WARP SPACING]` / `[WARP THICKNESS]` (and weft) | per-group `spacing` overrides |
| `[PRIVATE IWEAVEIT PROJECT]` | `extensions.weaveit` — carried verbatim, not described by the schema |
| `[PRIVATE WEAVEIT BASICS].WEAVETYPE` | `weaveType` — lower-cased and hyphenated (`DOUBLEWEAVE` → `"double-weave"`) |
| `[* SYMBOL *]`, `[* ZOOM]`, `[BITMAP *]`, `[TRANSLATIONS]`, `[DESIGN]` | dropped |

## Open questions

- **Colourways** (named alternate colour assignments so one draft renders in several colour stories)
  and **selvedges** (floating, doubled) — deferred. How would they slot in?
- **Rigid-heddle pickup**: pattern picked up by hand behind the heddle is not captured by
  `shed: up/down/neutral` — it needs a per-pick end list. Model it, or leave it to `extensions`?
- **`weaveType`** is a grab-bag: facing (`balanced` / `warp-faced` / `weft-faced`) and `profile` and
  `double-width` are different axes. Split it?
- **Multi-heddle rigid heddle**: is `shed` as an array (one entry per heddle) enough for 2- and
  3-heddle work?
- **File identity**: `.wif2` is a clean break. Is that better than reusing `.wif` and detecting JSON
  vs. INI by the first character?

## License

Copyright © 2026 Shannon Young. Licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), like the rest of this repository. Nothing
here claims any right in the WIF format itself.
