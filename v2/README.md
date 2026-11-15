# WIF 2

**Status:** draft proposal, not adopted · **Format:** JSON · **Extension:** `.wif2`

> Nothing here is decided, and no software should implement it yet. Comments go on
> [PR #1](https://github.com/WIF-Format/WIFStandard/pull/1).

WIF 2 is a proposed JSON successor to WIF 1.1. It stores a draft the way a weaver thinks about
it: named yarns, a warp and a weft built from runs and repeats of those yarns, and an explicit
loom type — floor loom, lift-plan loom, or rigid heddle. The JSON Schema is the specification.

| File | What it is |
| --- | --- |
| [Guide](wif-format_v2_guide.md) | Plain-language walkthrough of the format, for weavers and developers. |
| [`wif2.schema.json`](wif2.schema.json) | The specification: a JSON Schema (Draft 2020-12). A valid file validates against it. |
| [Proposal](proposal.md) | Why JSON, what changes from 1.1, how to convert a 1.1 file, and the open questions. |
| [`point-twill.wif2`](point-twill.wif2) | A complete file: a 4-shaft floor loom with tie-up and treadling. |
| [`lift-plan.wif2`](lift-plan.wif2) | A complete file: an 8-shaft dobby worked from a lift plan. |
| [`rigid-heddle-plaid.wif2`](rigid-heddle-plaid.wif2) | A complete file: a plaid on a rigid-heddle loom. |

[All versions](../README.md)
