# Final phase and common mistakes

## Final phase

When every app exports `org.wif-format.wif`, the transitional identifiers can be deleted. Each app then needs only the shared export, one document type that lists it, and, where it applies, its own conforming legacy type.

## Common mistakes

| Mistake | Effect |
|---|---|
| Private type also declares `public.filename-extension = wif` | The system must choose between competing claims; the winner changes with installs and updates. |
| Shared type conforms to a private type | Conflicting definitions across apps; possible loop in the type hierarchy. |
| Two `UTExportedTypeDeclarations` keys in one plist | The second silently replaces the first, so the shared type disappears. |
| `CFBundleTypeRank` instead of `LSHandlerRank` | The rank is ignored. |
| Typo such as an unclosed `<string>` tag | The plist fails to parse and the declarations are lost. |
| Exporting and importing the same identifier | Ambiguous ownership. |
| Scene-based iOS app without `scene:openURLContexts:` | App opens from Files but ignores the file. |
| Thumbnail extension not listing the shared type | Text icons in Files and Finder instead of draft thumbnails. |

Upgrading to the shared type is a few minutes of work, and it keeps your customers' files opening in your app. Take the time to unite as the original developers did in 1997.

<div class="github-only" markdown="1">

---

© 2026 the WIF-Format contributors. Licensed under [CC BY 4.0](../LICENSE). The WIF format is by Ravi Nielsen (Maple Hill Software) and the 1996–97 WIF developer groups; original text © 1996, 1997 Ravi Nielsen. The format itself is a shared standard, owned by no one.

</div>
