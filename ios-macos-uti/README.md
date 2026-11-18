# Developer Guide: Sharing the .wif Format on macOS and iOS

This page explains how a weaving application should declare the WIF file type in its `Info.plist` so that many apps can open the same `.wif` files on Apple platforms. It starts with the concepts, then gives the code for iOS and for macOS, and ends with common mistakes.

> **What has been tested.** The iOS declarations in this guide were tested in iWeaveIt on iPads running iOS 27, both with and without other weaving apps installed. The macOS declarations were tested in WeaveIt for Mac on a clean Mac with only WeaveIt, Fiberworks and TempoWeave installed. Both apps were then submitted to Apple for review. Please test on your own devices before release, and tell us what you find.

## Contents

| Section | Description |
| --- | --- |
| [The concepts](#the-concepts) | How Apple devices identify file types, and why WIF needs one shared type. |
| [iOS](#ios) | The `Info.plist` code for iPhone and iPad. |
| [macOS](#macos) | What differs on the Mac, and the `Info.plist` code. |
| [Common mistakes](#common-mistakes) | The mistakes to avoid on both platforms, and what each one does. |

## The concepts

### Why the extension is not enough

On Windows, a file association is a registry entry, and any number of programs can handle `.wif`. On macOS and iOS, the system identifies each file by a **Uniform Type Identifier (UTI)**, such as `public.jpeg` or `org.wif-format.wif`. The file extension is only a hint used to look up the UTI. Each app tells the system which UTIs it defines (exports) and which it can open, using keys in its `Info.plist`.

### The problem with a shared format

Apple expects each file format to have one owner. WIF has none; it was created in the 1990s by a committee of developers. When several apps each declare their own private UTI (for example `com.company.app.wif`) and each claims the `.wif` extension, the system has to choose one claim. The choice can change whenever any app is installed, updated, or removed. Users see this as another app suddenly becoming the default, thumbnails turning into plain text icons, or files appearing grayed out in other apps and in web file pickers.

### The solution: one shared type

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

### Rules that apply to every app

- **Conformance runs from the specific type up to the general type.** A private type conforms to `org.wif-format.wif`. The shared type must never conform to a private type, because that creates conflicting definitions or a loop.
- **Only `org.wif-format.wif` declares `public.filename-extension = wif`.** Private types must not repeat it.
- **Every app exports the shared type with exactly the same declaration.** When several apps export the same type, the system uses one of their declarations, and which one can change as apps are installed or updated. If every app's declaration is identical, it doesn't matter which one is used. If they differ, results are unpredictable.
- **Export a type you own; import a type you only reference. Never both for the same identifier.** Your own legacy type is exported, not imported.
- **Use the correct rank.** Use `Default` for the shared type, because no app owns it. Use `Owner` only for your own private identifiers. The key is `LSHandlerRank` (not `CFBundleTypeRank`).
- **One `UTExportedTypeDeclarations` key per plist.** All exported types go in a single array. A property list cannot hold the same key twice, and the second copy silently replaces the first.

### Legacy identifiers

The only legacy private identifier still in use for iWeaveIt is `com.canyonart.weaveit.wif` (and the older `com.canyonart.iweaveit.wif`, which only iWeaveIt needs). Because it conforms to `org.wif-format.wif`, other apps do **not** need to list it: a file typed with it is also a shared WIF file. On iOS, the weaving apps now use only the shared type. On the Mac, Fiberworks still uses its own private identifier; see the transitional notes for [macOS](#macos).

### Files that were never typed as WIF

Some apps export drafts as generic plain text and merely name them `.wif`. The system then treats the file as `public.plain-text`, which is a different branch of the type hierarchy, so apps that declare only `org.wif-format.wif` will not offer to open it. The correct fix belongs in the exporting app: it should write real WIF files typed as `org.wif-format.wif`. An app can add `public.plain-text` at `Alternate` rank as a workaround, but it will then appear in "Open With" for every text file, so we do not recommend it as a general rule.

### Do not set the default handler in code

Do not call `LSSetDefaultRoleHandlerForContentType` (macOS) at launch. It works inconsistently and overrides the user's own choice of default app. iOS has no public API to do this at all; the system and the user decide.

## iOS

On iOS every app is sandboxed, and the system, not the app, decides which app is the default opener. Declare the types correctly and do not try to force a default.

### Export the shared type (required)

```xml
<key>UTExportedTypeDeclarations</key>
<array>

    <!-- Shared canonical type. Copy exactly. -->
    <dict>
        <key>UTTypeIdentifier</key>
        <string>org.wif-format.wif</string>
        <key>UTTypeDescription</key>
        <string>Weaving Information File</string>
        <key>UTTypeConformsTo</key>
        <array>
            <string>public.data</string>
        </array>
        <key>UTTypeTagSpecification</key>
        <dict>
            <key>public.filename-extension</key>
            <array>
                <string>wif</string>
            </array>
        </dict>
    </dict>

    <!-- Only if your app has its own legacy identifier.
         Same array as above. No extension tag. -->
    <dict>
        <key>UTTypeIdentifier</key>
        <string>com.yourcompany.yourapp.wif</string>
        <key>UTTypeDescription</key>
        <string>YourApp WIF File</string>
        <key>UTTypeConformsTo</key>
        <array>
            <string>org.wif-format.wif</string>
        </array>
    </dict>

</array>
```

### Register the document types (required)

```xml
<key>CFBundleDocumentTypes</key>
<array>

    <!-- The shared type -->
    <dict>
        <key>CFBundleTypeName</key>
        <string>Weaving Information File</string>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>     <!-- Viewer if your app only reads WIF -->
        <key>LSHandlerRank</key>
        <string>Default</string>
        <key>LSItemContentTypes</key>
        <array>
            <string>org.wif-format.wif</string>
        </array>
    </dict>

    <!-- Only if you have your own legacy identifier -->
    <dict>
        <key>CFBundleTypeName</key>
        <string>YourApp WIF File</string>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>LSHandlerRank</key>
        <string>Owner</string>
        <key>LSItemContentTypes</key>
        <array>
            <string>com.yourcompany.yourapp.wif</string>
        </array>
    </dict>

</array>
```

### Other keys that matter on iOS

```xml
<!-- Opens files from Files / iCloud Drive in place -->
<key>LSSupportsOpeningDocumentsInPlace</key>
<true/>
```

> **Apps that use scenes must handle incoming files in the scene delegate.** If your plist has `UIApplicationSceneManifest`, iOS does not call `application:openURL:options:`. Implement `scene:openURLContexts:` (app already running) and read `connectionOptions.URLContexts` in `scene:willConnectToSession:options:` (cold launch). Otherwise the app launches from Files and silently ignores the file.

**Thumbnail and Quick Look extensions** have their own `Info.plist`. Their `QLSupportedContentTypes` list must include `org.wif-format.wif`, or the extension will not run for files typed with the shared identifier.

### No transitional identifiers on iOS

On iOS, the weaving apps that once had their own identifiers (iWeaveIt and TempoWeave) now export `org.wif-format.wif`. Your shared document type only needs to list `org.wif-format.wif`, as in [Register the document types](#register-the-document-types-required).

If the check in [Check your result on a device](#check-your-result-on-a-device) shows a different identifier, another installed app is still claiming the `.wif` extension with its own type. Contact that developer and point them to this guide.

### Check your result on a device

Ask the system what it assigns to the extension. With only your app installed, and again after installing other weaving apps, it should print `org.wif-format.wif`.

```objc
// Objective-C (link UniformTypeIdentifiers.framework, iOS 14+)
UTType *t = [UTType typeWithFilenameExtension:@"wif"];
NSLog(@"System type for .wif: %@", t.identifier);
```

```swift
// Swift
import UniformTypeIdentifiers
print(UTType(filenameExtension: "wif")?.identifier ?? "none")
```

If another identifier appears, another installed app is still claiming the extension with its own type. Contact that developer and point them to this page.

## macOS

The type declarations are the same as on iOS: the same shared type, the same conformance rules, and the same exact copy. The differences are in what else surrounds them.

| Topic | macOS |
|---|---|
| Sandbox | Required only for the Mac App Store. Apps distributed directly (Developer ID) need not be sandboxed. Sandbox permissions such as `com.apple.security.files.user-selected.read-write` go in the entitlements file, not the `Info.plist`. |
| Default handler | Users can choose in Finder (Get Info, Open with, Change All). Do not call `LSSetDefaultRoleHandlerForContentType` to override it. |
| Document class | Document-based apps may add `NSDocumentClass` to each document type entry. |
| Incoming files | Handled by `NSDocument` or `application:openURLs:`. The scene-delegate rule in [Other keys that matter on iOS](#other-keys-that-matter-on-ios) is iOS only. |
| Transitional list | The Mac version of Fiberworks still declares its own private identifier, `com.fiberworks-pcw.wif`. Keep it in your shared document type until Fiberworks updates. TempoWeave already uses the shared type. |

### Export the shared type (same as iOS)

Use exactly the same declaration as for iOS, in [Export the shared type](#export-the-shared-type-required). The identifier, description, `public.data` conformance and extension tag must match.

### Register the document types (macOS)

```xml
<key>CFBundleDocumentTypes</key>
<array>

    <!-- The shared type, plus the older Fiberworks
         identifier during the transition -->
    <dict>
        <key>CFBundleTypeName</key>
        <string>Weaving Information File</string>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>LSHandlerRank</key>
        <string>Default</string>
        <key>LSItemContentTypes</key>
        <array>
            <string>org.wif-format.wif</string>
            <string>com.fiberworks-pcw.wif</string>   <!-- transitional -->
        </array>
        <key>NSDocumentClass</key>
        <string>YourDocument</string>   <!-- optional -->
    </dict>

    <!-- Your own legacy identifier, if you have one -->
    <dict>
        <key>CFBundleTypeName</key>
        <string>YourApp WIF File</string>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>LSHandlerRank</key>
        <string>Owner</string>
        <key>LSItemContentTypes</key>
        <array>
            <string>com.yourcompany.yourapp.wif</string>
        </array>
    </dict>

</array>
```

### Check your result on a Mac

In Terminal, ask for the type of a `.wif` file. After the shared type is in place, `kMDItemContentType` should show `org.wif-format.wif`. If it shows another identifier, including an older private type of your own, an installed app (perhaps an older version or build of yours) is still claiming the `.wif` extension with its own type. Check `kMDItemContentTypeTree`: if `org.wif-format.wif` is not in it, that type does not conform to the shared one.

```
mdls -name kMDItemContentType -name kMDItemContentTypeTree yourfile.wif
```

### Final phase

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

---

WIF Specification © 1997 Maple Hill Software
wif-format.org domain name — initially maintained by Canyon Art, LLC
