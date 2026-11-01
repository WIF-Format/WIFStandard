# Weaving Information File (WIF)

The **Weaving Information File** is an open, plain-text format that lets weaving programs and
computerized looms exchange drafts — the threading, tie-up, treadling or lift plan, and colors
needed to reproduce a piece of cloth. It was created in 1996 and revised in 1997, and every
major weaving program still reads and writes it.

This site documents the format in plain language for two readers at once: a **weaver**, who
knows what a threading or a tie-up is but not how it is stored, and a **developer**, who needs
to read or write the file correctly. Each version has a guide, a specification, and example
files.

---

## Versions

| Version | Status | Format | Guide | Specification | Examples |
| --- | --- | --- | --- | --- | --- |
| **[1.1](v1.1/)** | Current standard (1997) | INI text, `.wif` | [Guide](v1.1/wif-format_v1_1_guide.md) | [Spec](v1.1/wif-format_v1_1.md) | [`.wif`](v1.1/twill-color-and-weave.wif) |
| **[1.2](https://github.com/WIF-Format/WIFStandard/tree/wif1.2-proposal/v1.2)** | Draft proposal | INI text, `.wif` | [Guide](https://github.com/WIF-Format/WIFStandard/blob/wif1.2-proposal/v1.2/wif-format_v1_2_guide.md) | [Spec](https://github.com/WIF-Format/WIFStandard/blob/wif1.2-proposal/v1.2/wif-format_v1_2.md) | [folder](https://github.com/WIF-Format/WIFStandard/tree/wif1.2-proposal/v1.2) |
| **[2](https://github.com/WIF-Format/WIFStandard/tree/wif2-proposal/v2)** | Draft proposal | JSON, `.wif2` | [Guide](https://github.com/WIF-Format/WIFStandard/blob/wif2-proposal/v2/wif-format_v2_guide.md) | [Schema](https://github.com/WIF-Format/WIFStandard/blob/wif2-proposal/v2/wif2.schema.json) | [folder](https://github.com/WIF-Format/WIFStandard/tree/wif2-proposal/v2) |

**Which one should I use?** Read and write **WIF 1.1**. It is the only version software
supports today. The two proposals are open for discussion and nothing in them is decided:

- **WIF 1.2** stays in the INI text format, standardizes the extensions that already ship in
  the field, and formally retires the parts nothing implements. Every 1.1 file is a valid 1.2
  file. Comments go on [PR #2](https://github.com/WIF-Format/WIFStandard/pull/2).
- **WIF 2** is a larger rewrite to JSON, with a published schema and a normalized data model.
  Comments go on [PR #1](https://github.com/WIF-Format/WIFStandard/pull/1).

Each version folder holds the same kinds of file:

| File | What it is |
| --- | --- |
| `README.md` | The version's landing page. |
| `wif-format_v*_guide.md` | Plain-language guide: what each part means at the loom, then how it is encoded. |
| `wif-format_v*.md` | Reference specification: every section and key, in tables. WIF 2 uses its JSON Schema instead. |
| `*.wif`, `*.wif2` | Complete example files. |
| `proposal.md` | Draft versions only: why each change is proposed, and open questions. |

---

## Weaving terms

These terms are used the same way in every version.

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
| **sett** | How closely the ends are spaced, usually in ends per inch (EPI). Picks per inch (PPI) is the weft equivalent. |
| **drawdown** | The picture of the finished cloth produced by combining threading, tie-up / lift plan, and treadling. |

---

## File types

| Extension | Versions | Platform type identifier |
| --- | --- | --- |
| `.wif` | 1.1, 1.2 | `org.wif-format.wif` |
| `.wif2` | 2 (proposed) | — |

On Windows a file extension is enough. Apple platforms identify file types by Uniform Type
Identifier, so every app that reads or writes WIF should declare the same shared one — see
**[Sharing the `.wif` File Type on macOS and iOS](apple-uti-setup.md)**.

---

## For AI agents

[`llms.txt`](llms.txt) lists every page on this site with a one-line description, following
the [llms.txt](https://llmstxt.org/) convention. Every page is Markdown and every example is a
complete file, so either can be fetched and read directly.

---

## History and credits

WIF was written by Ravi Nielsen (Maple Hill Software). The 1.0 and 1.1 developer groups
included Bob Keates (Fiberworks), Rob Sinkler (Swiftweave), Dana Cartwright (WeaveMaker),
Bjørn Myhre (WeavePoint), Jane Eisenstein, Sally Breckenridge (WeaveIt), Mark Kloosterman
and Dini Cameron (ProWeave), Mitch Eatough, and Peter Straus (AVL).

The format has not changed since version 1.1 in 1997. The original WIF 1.1 text,
[Copyright © 1996, 1997 Ravi Nielsen](http://www.tantradharma.com/maplehill/wif/wif1-1.txt),
remains the historical source; nothing on this site reproduces it.

More about the format and the software that supports it: [wif-format.org](https://wif-format.org).

---

## License and attribution

**The WIF format** is the work of Ravi Nielsen (Maple Hill Software) and the 1996 and 1997
WIF developer groups — see [History and credits](#history-and-credits). Nothing on this site
claims any right in the format itself, and no one owns it; it is a shared standard.

**These documents** are independent, plain-language descriptions of that format. They do not
copy or adapt the original specification.

Copyright © 2026 Shannon Young. Licensed under the
[Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)
(CC BY 4.0). You are free to copy, translate, adapt, and redistribute them, for any purpose
including commercially, as long as you give appropriate credit and indicate any changes.
See [`LICENSE`](LICENSE).
