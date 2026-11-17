# Part 2. iOS

On iOS every app is sandboxed, and the system, not the app, decides which app is the default opener. Declare the types correctly and do not try to force a default.

## 2.1 Export the shared type (required)

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

## 2.2 Register the document types (required)

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

## 2.3 Other keys that matter on iOS

```xml
<!-- Opens files from Files / iCloud Drive in place -->
<key>LSSupportsOpeningDocumentsInPlace</key>
<true/>
```

> **Apps that use scenes must handle incoming files in the scene delegate.** If your plist has `UIApplicationSceneManifest`, iOS does not call `application:openURL:options:`. Implement `scene:openURLContexts:` (app already running) and read `connectionOptions.URLContexts` in `scene:willConnectToSession:options:` (cold launch). Otherwise the app launches from Files and silently ignores the file.

**Thumbnail and Quick Look extensions** have their own `Info.plist`. Their `QLSupportedContentTypes` list must include `org.wif-format.wif`, or the extension will not run for files typed with the shared identifier.

## 2.4 Transitional notes for iOS

Older releases of some weaving apps declared their own private identifiers with the `.wif` extension. While those versions remain installed, the system may resolve `.wif` to the older identifier, and your app will not list itself for it. If you see this, add the older identifier to the `LSItemContentTypes` of your shared document type:

```xml
<string>org.wif-format.wif</string>
<string>com.fiberworks-pcw.wif</string>   <!-- older builds only -->
<string>com.loftyfiber.wif</string>       <!-- older builds only -->
```

Remove these lines once those apps have shipped the shared type and users have updated.

## 2.5 Check your result on a device

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

Next: [Part 3. macOS](macos.md)

---

WIF Specification © 1997 Maple Hill Software
wif-format.org domain name — initially maintained by Canyon Art, LLC
