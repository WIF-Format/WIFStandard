# Proposals under discussion

> **Nothing here is decided, and no weaving program should use these proposals yet.** Read
> and write [WIF 1.1](v1.1/README.md), the current standard.

Weaving software developers agree that WIF could be improved, but they have different ideas
about how. Two ideas for the next version are written up here so that everyone can read them
and comment. Each one has a guide, a reference specification, a page explaining the reasons
for each change, and example files.

| Proposal | What it does | Read it | Discuss it |
| --- | --- | --- | --- |
| **WIF 1.2** | A small update. Files stay plain text with the `.wif` extension, like WIF 1.1. It adds the extra information that weaving programs already write today, such as a yarn list and repeats, and marks as deprecated the parts no current program uses. Every WIF 1.1 file is also a valid WIF 1.2 file. | [WIF 1.2](https://developer.wif-format.org/v1.2/) | [Pull request 2](https://github.com/WIF-Format/WIFStandard/pull/2) |
| **WIF 2** | A larger redesign. Files use JSON, a common text format that programs read easily, with a new `.wif2` extension. A formal description of the file (a "schema") lets programs check that a file is correct. | [WIF 2](https://developer.wif-format.org/v2/) | [Pull request 1](https://github.com/WIF-Format/WIFStandard/pull/1) |

## How to comment

Each proposal is discussed on GitHub, the website where these documents are kept. A *pull
request* is a GitHub page that shows a proposed change to the documents, with a place to
discuss it before anyone accepts it.

To comment on a proposal:

1. Open its pull request from the table above.
2. Sign in to GitHub. An account is free.
3. Write your comment in the box at the bottom of the page.

If you would rather not use GitHub, email your comments to
[info@wif-format.org](mailto:info@wif-format.org).

The proposal pages on this website always show the latest version of each proposal.

<div class="github-only" markdown="1">

---

© 2026 the WIF-Format contributors. Licensed under [CC BY 4.0](LICENSE). The WIF format is by Ravi Nielsen (Maple Hill Software) and the 1996–97 WIF developer groups; original text © 1996, 1997 Ravi Nielsen. The format itself is a shared standard, owned by no one.

</div>
