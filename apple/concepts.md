# Part 1. The concepts

## Why the extension is not enough

On Windows, a file association is a registry entry, and any number of programs can handle `.wif`. On macOS and iOS, the system identifies each file by a **Uniform Type Identifier (UTI)**, such as `public.jpeg` or `org.wif-format.wif`. The file extension is only a hint used to look up the UTI. Each app tells the system which UTIs it defines (exports) and which it can open, using keys in its `Info.plist`.

## The problem with a shared format

Apple expects each file format to have one owner. WIF has none; it was created in the 1990s by a committee of developers. When several apps each declare their own private UTI (for example `com.company.app.wif`) and each claims the `.wif` extension, the system has to choose one claim. The choice can change whenever any app is installed, updated, or removed. Users see this as another app suddenly becoming the default, thumbnails turning into plain text icons, or files appearing grayed out in other apps and in web file pickers.

## The solution: one shared type

All apps export the same canonical type, `org.wif-format.wif`, and only that type claims the `.wif` extension. An app that needs its own private identifier for legacy reasons keeps it, but makes it *conform to* the shared type and does not claim the extension.

```
                  public.data
                       |
              org.wif-format.wif
        (shared canonical type; the only one
            that claims the .wif extension)
                       |
          +------------+-------------+
          |                          |
com.canyonart.weaveit.wif     (any other app's
   (legacy private type)       legacy private type)
```

## Rules that apply to every app

- **Conformance runs from the specific type up to the general type.** A private type conforms to `org.wif-format.wif`. The shared type must never conform to a private type, because that creates conflicting definitions or a loop.
- **Only `org.wif-format.wif` declares `public.filename-extension = wif`.** Private types must not repeat it.
- **Every app exports the shared type with exactly the same declaration.** The system merges identical declarations from different apps. Different definitions cause unpredictable results.
- **Export a type you own; import a type you only reference. Never both for the same identifier.** Your own legacy type is exported, not imported.
- **Use the correct rank.** Use `Default` for the shared type, because no app owns it. Use `Owner` only for your own private identifiers. The key is `LSHandlerRank` (not `CFBundleTypeRank`).
- **One `UTExportedTypeDeclarations` key per plist.** All exported types go in a single array. A property list cannot hold the same key twice, and the second copy silently replaces the first.

## Legacy identifiers

The only legacy private identifier still in use for iWeaveIt is `com.canyonart.weaveit.wif` (and the older `com.canyonart.iweaveit.wif`, which only iWeaveIt needs). Because it conforms to `org.wif-format.wif`, other apps do **not** need to list it: a file typed with it is also a shared WIF file. Some older versions of other apps used their own private identifiers; see the transitional notes in [Part 2](ios.md#24-transitional-notes-for-ios) and [Part 3](macos.md).

## Files that were never typed as WIF

Some apps export drafts as generic plain text and merely name them `.wif`. The system then treats the file as `public.plain-text`, which is a different branch of the type hierarchy, so apps that declare only `org.wif-format.wif` will not offer to open it. The correct fix belongs in the exporting app: it should write real WIF files typed as `org.wif-format.wif`. An app can add `public.plain-text` at `Alternate` rank as a workaround, but it will then appear in "Open With" for every text file, so we do not recommend it as a general rule.

## Do not set the default handler in code

Do not call `LSSetDefaultRoleHandlerForContentType` (macOS) at launch. It works inconsistently and overrides the user's own choice of default app. iOS has no public API to do this at all; the system and the user decide.

Next: [Part 2. iOS](ios.md)

---

WIF Specification © 1997 Maple Hill Software
wif-format.org domain name — initially maintained by Canyon Art, LLC
