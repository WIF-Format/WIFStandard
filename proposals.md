# Proposals under discussion

> **Nothing here is decided, and no software should implement these proposals.** Read and
> write [WIF 1.1](v1.1/README.md), the current standard.

Two ways forward from WIF 1.1 are open for discussion. Each one has a guide, a reference
specification, a proposal page explaining the reasons for each change, and example files.
Comments go on the proposal's pull request.

| Proposal | Format | What it does | Read it | Comment |
| --- | --- | --- | --- | --- |
| **WIF 1.2** | INI text, `.wif` | Keeps the 1.1 format and standardizes the extensions that already ship in the field. Formally deprecates the parts no current program implements. Every 1.1 file is a valid 1.2 file. | [WIF 1.2](https://developer.wif-format.org/v1.2/) | [PR #2](https://github.com/WIF-Format/WIFStandard/pull/2) |
| **WIF 2** | JSON, `.wif2` | A larger rewrite to JSON, with a published schema and a normalized data model. | [WIF 2](https://developer.wif-format.org/v2/) | [PR #1](https://github.com/WIF-Format/WIFStandard/pull/1) |

The proposal files are kept on their own branches,
[`wif1.2-proposal`](https://github.com/WIF-Format/WIFStandard/tree/wif1.2-proposal/v1.2) and
[`wif2-proposal`](https://github.com/WIF-Format/WIFStandard/tree/wif2-proposal/v2), and the
website is rebuilt whenever either branch changes.

<div class="github-only" markdown="1">

---

© 2026 the WIF-Format contributors. Licensed under [CC BY 4.0](LICENSE). The WIF format is by Ravi Nielsen (Maple Hill Software) and the 1996–97 WIF developer groups; original text © 1996, 1997 Ravi Nielsen. The format itself is a shared standard, owned by no one.

</div>
