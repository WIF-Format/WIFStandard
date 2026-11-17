# Developer Guide: Sharing the .wif Format on macOS and iOS

This page explains how a weaving application should declare the WIF file type in its `Info.plist` so that many apps can open the same `.wif` files on Apple platforms. [Part 1](concepts.md) explains the concepts, [Part 2](ios.md) gives the iOS code, and [Part 3](macos.md) gives the macOS code.

> **What has been tested.** The iOS declarations below were tested in iWeaveIt on iPads running iOS 27, both with and without other weaving apps installed. The macOS declarations are the transitional scheme previously published on this site, which has worked in iWeaveIt for Mac. Please test on your own devices before release, and tell us what you find.

| Part | What it covers |
| --- | --- |
| [Part 1. The concepts](concepts.md) | Why the extension is not enough, the shared type, and the rules every app follows. |
| [Part 2. iOS](ios.md) | `Info.plist` declarations for iOS, other keys that matter, and how to check the result on a device. |
| [Part 3. macOS](macos.md) | What differs on macOS, the document types, and how to check the result on a Mac. |
| [Final phase and common mistakes](final-phase.md) | What changes once every app has adopted the shared type, and the mistakes to avoid. |

---

WIF Specification © 1997 Maple Hill Software
wif-format.org domain name — initially maintained by Canyon Art, LLC
