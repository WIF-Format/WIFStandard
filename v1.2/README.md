# WIF 1.2

**Status:** draft proposal, not adopted · **Format:** INI text · **Extension:** `.wif`

> Nothing here is decided, and no software should implement it yet. Comments go on
> [PR #2](https://github.com/WIF-Format/WIFStandard/pull/2).

WIF 1.2 is WIF 1.1 with additions: a yarn table, repeat blocks, an explicit loom type
(including rigid heddle), UTF-8 text, and ISO dates. Every 1.1 file is a valid 1.2 file, and a
1.1 program can still read a 1.2 file.

| File | What it is |
| --- | --- |
| [Guide](wif-format_v1_2_guide.md) | Plain-language walkthrough of the whole format, with each change from 1.1 marked. |
| [Specification](wif-format_v1_2.md) | Reference tables: every section and key, with type and notes. |
| [Proposal](proposal.md) | Why each change is proposed, and the open questions. |
| [`twill-color-and-weave.wif`](twill-color-and-weave.wif) | A complete file: a 4-shaft floor-loom twill with a yarn table and a repeat. |
| [`rigid-heddle-plain.wif`](rigid-heddle-plain.wif) | A complete file: plain weave on a single-heddle rigid-heddle loom. |

[All versions](../README.md)
