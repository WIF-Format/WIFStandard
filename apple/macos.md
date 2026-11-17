# Part 3. macOS

The type declarations are the same as on iOS: the same shared type, the same conformance rules, and the same exact copy. The differences are in what else surrounds them.

| Topic | macOS |
|---|---|
| Sandbox | Required only for the Mac App Store. Apps distributed directly (Developer ID) need not be sandboxed. Sandbox permissions such as `com.apple.security.files.user-selected.read-write` go in the entitlements file, not the `Info.plist`. |
| Default handler | Users can choose in Finder (Get Info, Open with, Change All). Do not call `LSSetDefaultRoleHandlerForContentType` to override it. |
| Document class | Document-based apps may add `NSDocumentClass` to each document type entry. |
| Incoming files | Handled by `NSDocument` or `application:openURLs:`. The scene-delegate rule in [section 2.3](ios.md#23-other-keys-that-matter-on-ios) is iOS only. |
| Transitional list | Mac versions of other apps may still declare private identifiers. Keep the older identifiers in your shared document type until those apps update. |

## 3.1 Export the shared type (required)

Use exactly the same declaration as in [section 2.1](ios.md#21-export-the-shared-type-required). The identifier, description, `public.data` conformance and extension tag must match.

## 3.2 Register the document types (macOS)

```xml
<key>CFBundleDocumentTypes</key>
<array>

    <!-- The shared type, plus older private identifiers
         from other apps during the transition -->
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
            <string>com.loftyfiber.wif</string>       <!-- transitional -->
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

## 3.3 Check your result on a Mac

In Terminal, ask for the type of a `.wif` file. After the shared type is in place it should show `org.wif-format.wif` (or your own legacy type, which conforms to it):

```
mdls -name kMDItemContentType -name kMDItemContentTypeTree yourfile.wif
```

Next: [Final phase and common mistakes](final-phase.md)

<div class="github-only" markdown="1">

---

© 2026 the WIF-Format contributors. Licensed under [CC BY 4.0](../LICENSE). The WIF format is by Ravi Nielsen (Maple Hill Software) and the 1996–97 WIF developer groups; original text © 1996, 1997 Ravi Nielsen. The format itself is a shared standard, owned by no one.

</div>
