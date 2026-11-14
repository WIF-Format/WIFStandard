# Sharing the `.wif` File Type on macOS and iOS

A guide for developers setting up an Xcode project so that their app opens and saves
`.wif` files **as a shared file type**. All weaving apps on the system can then open a WIF
file, and the user stays in control of which app opens it by default.

_Originally written by Sally Breckenridge (Canyon Art WeaveIt) for
[wif-format.org](https://wif-format.org/)._

---

## Overview

On a Mac the user sees a file extension, much as on Windows. Behind the scenes it is more
complicated, on both macOS and iOS. Apple identifies every file type with a **Uniform Type Identifier
(UTI)**, a reverse-DNS name such as `com.example.myapp.document`. An app *declares* the
UTIs it understands in its `Info.plist`. Launch Services, the part of the system that
decides which app opens a file, works from those UTIs, not from the extension.

Apple expects each UTI, and the extension it claims, to have one owner. No single company
owns WIF, so it doesn't fit that model. When several apps each declare their own private
UTI and claim `.wif` for it, Launch Services has to pick a winner. The result is
unpredictable: it can change when an app is updated or a new app is installed. Users are
then pushed to delete the apps that "lost".

The fix is for every app to agree on **one shared UTI for WIF: `org.wif-format.wif`**.
With a single UTI, every app can open and save the format. On a Mac the user chooses the
default through Finder's **Get Info → Open with → Change All…**, as they would for any
other shared type. On iOS and iPadOS there is no default to set: the user picks an app
from the share sheet or from the Files app (see [iOS and iPadOS](#ios-and-ipados)).

Several apps already shipped with their own private WIF UTI, so we get there in two
phases:

1. **Transitional phase.** Declare the shared UTI. Keep your legacy private UTI, but make
   it conform to the shared one. List the other apps' legacy UTIs so you can still open
   their files.
2. **Final phase.** Once every app declares the shared UTI, drop the private and legacy
   UTIs and declare only `org.wif-format.wif`.

---

## UTI hierarchy

During the transition, the declarations form this hierarchy. The private legacy UTIs
exist only to support existing customers until every app adopts the shared canonical UTI.
In the final phase that middle layer goes away.

```
                        public.data
                             │
                             ▼
                    org.wif-format.wif
                  (shared canonical UTI)
                             │
        ┌────────────────────┼────────────────────┐   … more apps
        ▼                    ▼                    ▼
com.canyonart.weaveit.wif  com.fiberworks-pcw.wif  com.loftyfiber.wif
  (private legacy UTI)     (private legacy UTI)    (private legacy UTI)
        │                    │                    │
        └─────────────── conforms to ─────────────┘
```

- `public.data` is Apple's base type for data files.
- `org.wif-format.wif` is the single shared canonical UTI for `.wif` files.
- Each app keeps its private UTI for backward compatibility.
- Every private UTI **conforms to** the shared UTI.
- **Only the shared UTI declares the `.wif` extension.**
- This structure prevents Launch Services conflicts and keeps file associations stable.

---

## Editing `Info.plist` in Xcode

The declarations below are the minimal set needed for sharing. Add any other keys your
app uses.

To edit the XML directly, right-click `Info.plist` in the Project navigator and choose
**Open As → Source Code**. If your target has no `Info.plist` file (newer Xcode
templates generate it from build settings), use the target's **Info** tab instead:
**Document Types**, **Exported Type Identifiers**, and **Imported Type Identifiers** each
have their own editor there, with fields matching the keys below.

| Key | What it does |
| --- | --- |
| `CFBundleDocumentTypes` | The document types your app can open, and in what role (`Editor` or `Viewer`). |
| `LSItemContentTypes` | Inside a document type: the UTIs that the type covers. |
| `LSHandlerRank` | Inside a document type: how strongly your app claims it: `Owner`, `Default`, `Alternate`, or `None`. Always set it; don't rely on a default. |
| `UTExportedTypeDeclarations` | UTIs your app defines, so the system knows them on any device where your app is installed. |
| `UTImportedTypeDeclarations` | UTIs another app defines that you rely on. |

> **Export or import, never both.** Do not import a UTI you also export, and do not import
> your own UTI.

### Why every app exports the same declaration

Apple's model is that one app owns and **exports** a type, and every other app that uses it
**imports** it. WIF has no owner, so every WIF app exports `org.wif-format.wif`. Apple
doesn't document what happens when several installed apps export the same identifier.
Developers report that Launch Services quietly uses one of the declarations, and which
one can change as apps are installed and updated.

That is harmless only if every declaration is the same. So the shared declaration below
must be copied **exactly as written**: the same description, the same conformance, and the
same extension list, in every app. If one app adds an icon, a reference URL, an uppercase
`WIF`, or a different `UTTypeConformsTo`, then how `.wif` files behave can depend on which
app was installed last. To propose a change to the shared declaration, open an issue on
this repository so every WIF app can adopt it together.

---

## Transitional phase

Apps still ship with legacy private UTIs.

**Goal:** avoid breaking existing installs while moving everyone to the shared UTI.

1. **Export the shared UTI** `org.wif-format.wif`, and declare the `wif` extension on it.
2. **Keep exporting your legacy private UTI**, but do **not** declare the `.wif`
   extension on it.
3. **Make your private UTI conform to** `org.wif-format.wif`. Files don't carry a UTI:
   the system works out a file's type from its extension each time. Once your private
   UTI no longer claims `.wif`, no file resolves to it. Keeping it declared and conforming
   means that anything still referring to it is recognized as WIF: your own code, and
   other apps' document-type lists.
4. **In `CFBundleDocumentTypes`, list the other apps' private UTIs** in the shared type's
   `LSItemContentTypes`. Then your app appears in **Open With** for WIF files that an
   older version of another app has claimed. You don't need to list your own private UTI
   there; it has its own document type entry.

The private UTIs to list today are:

- `com.canyonart.weaveit.wif`
- `com.fiberworks-pcw.wif`
- `com.loftyfiber.wif`

When another app ships with the shared UTI, its private UTI no longer needs listing. When
a new WIF app ships on the Mac with its own private UTI, the list grows for everyone.
Please open an issue or pull request on this repository to update it.

> **Do not set your app as the default for WIF in code.** Don't call
> `LSSetDefaultRoleHandlerForContentType` (deprecated since macOS 12) or its replacement,
> `NSWorkspace.setDefaultApplication(at:toOpenContentType:)`, for example in
> `applicationDidFinishLaunching`. Either one overrides the user's own choice of which
> app opens their files, and the older call works only some of the time.

### Transitional `Info.plist`

Replace `com.yourcompany.yourapp.wif` and the descriptions with your own. Entries marked
`TEMPORARY` are removed in the final phase.

```xml
<!-- ============================================================ -->
<!--  DOCUMENT TYPES                                              -->
<!--  Associates file types with your app in Finder.              -->
<!--  This shows only the .wif types; add entries for your app's  -->
<!--  own private formats. On macOS you may also add              -->
<!--  NSDocumentClass to name the class that reads the file.      -->
<!-- ============================================================ -->
<key>CFBundleDocumentTypes</key>
<array>
    <!-- The shared WIF format -->
    <dict>
        <key>CFBundleTypeName</key>
        <string>Weaving Information File</string>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>          <!-- Viewer if your app only reads WIF -->
        <key>LSHandlerRank</key>
        <string>Default</string>         <!-- shared type: nobody is Owner -->
        <key>LSItemContentTypes</key>
        <array>
            <string>org.wif-format.wif</string>
            <string>com.canyonart.weaveit.wif</string>  <!-- TEMPORARY -->
            <string>com.fiberworks-pcw.wif</string>     <!-- TEMPORARY -->
            <string>com.loftyfiber.wif</string>         <!-- TEMPORARY -->
        </array>
    </dict>

    <!-- Your app's private WIF UTI (TEMPORARY) -->
    <dict>
        <key>CFBundleTypeName</key>
        <string>YourApp WIF File</string>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>LSHandlerRank</key>
        <string>Owner</string>           <!-- you own your private UTI -->
        <key>LSItemContentTypes</key>
        <array>
            <string>com.yourcompany.yourapp.wif</string>
        </array>
    </dict>

    <!-- Add your app's other private formats here -->
</array>

<!-- ============================================================ -->
<!--  EXPORTED TYPE DECLARATIONS                                  -->
<!--  UTIs your app defines. Both go in ONE array: a key may      -->
<!--  appear only once in a plist dictionary, so do not add a     -->
<!--  second UTExportedTypeDeclarations key.                      -->
<!-- ============================================================ -->
<key>UTExportedTypeDeclarations</key>
<array>
    <!-- The shared WIF UTI. Include this block exactly as written. -->
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

    <!-- Your app's private WIF UTI (TEMPORARY).                  -->
    <!-- Conforms to the shared UTI. Does NOT declare .wif itself. -->
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

---

## Final phase

Once every app exports `org.wif-format.wif`, things get much simpler. The only document
type you need is the shared format, and the only UTI you need to export is the shared
one. Everyone uses the same file type.

Old `.wif` files need nothing extra. A file doesn't remember which app's UTI it was saved
under, so every `.wif` file resolves to `org.wif-format.wif` once no installed app claims
the extension for a private UTI. Legacy UTIs only matter while an old version of some app
is still installed. Keep listing them for as long as that's likely for your users.

Start from the transitional example and remove every `TEMPORARY` entry. You're left with:

```xml
<key>CFBundleDocumentTypes</key>
<array>
    <!-- The shared WIF format -->
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
        </array>
    </dict>
</array>

<key>UTExportedTypeDeclarations</key>
<array>
    <!-- The shared WIF UTI -->
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
</array>
```

---

## iOS and iPadOS

The same `CFBundleDocumentTypes` and `UTExportedTypeDeclarations` entries apply. iOS
apps also need these keys to work with `.wif` files in the Files app:

| Key | Value | What it does |
| --- | --- | --- |
| `LSSupportsOpeningDocumentsInPlace` | `true` | Opens the file where it is, in Files or iCloud Drive, instead of a copy. |
| `UISupportsDocumentBrowser` | `true` | Only if your app is built on a document browser (`UIDocumentBrowserViewController`, or SwiftUI `DocumentGroup`). |

iOS has no Finder **Get Info**, and no default app for a file type. The user picks an app
each time from the share sheet, or from **Open in** in the Files app. Every app that
declares the shared UTI is offered there.

---

## Final thoughts

_From Sally Breckenridge:_

> I submitted an app using the transitional format, and it works. There is still a lot of
> uncertainty about how Apple's Launch Services decides which app owns a file type. As
> things stand, only one app wins. That isn't really a win, because customers are then
> forced to choose one version and delete the others, and newer apps tend to lose that
> choice because people don't like change. Even if your app is the winner today, a new app
> may launch tomorrow and change that. Take the time to unite, as the WIF developers did
> in 1997.

---

Return to the [WIF format guide](README.md).
