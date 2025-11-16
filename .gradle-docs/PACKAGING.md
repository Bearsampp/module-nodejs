### Packaging and Archive Layout

This module’s Gradle build produces archives that include the top-level version folder, matching the behavior used by the Bruno module.

Example structure for version 24.6.0:

```
bearsampp-nodejs-24.6.0-2025.8.21.7z
└─ nodejs24.6.0/
   ├─ node.exe
   ├─ npm
   ├─ etc/
   └─ ...
```

Key implementation detail in `build.gradle`:

- The compression is run from the parent directory of the prep folder and the folder name (e.g. `nodejs24.6.0`) is passed explicitly to the archiver. This guarantees the archive contains the version folder at the root.

7‑Zip flow:

```
ProcessBuilder([7z.exe, 'a', '-t7z', <archive>, <folderName>])
  .directory(<parentDirOfFolder>)
```

Zip flow (Gradle `Zip` task):

```
from(prepPath.parentFile) {
    include "${prepPath.name}/**"
}
```

### How to verify locally

- Build a release for a specific version:
  - `gradle release -PbundleVersion=24.6.0`

- Locate the output archive under:
  - `<bearsampp-build>/<type>/<name>/<bundle.release>/`
  - Example: `../bearsampp-build/bins/nodejs/2025.8.21/bearsampp-nodejs-24.6.0-2025.8.21.7z`

- Verify with 7‑Zip (Windows):
  - Command prompt: `7z l bearsampp-nodejs-24.6.0-2025.8.21.7z | findstr nodejs24.6.0`
  - You should see entries starting with `nodejs24.6.0/`.

- Verify with any Zip tool (if `bundle.format=zip`):
  - The first entry should be `nodejs<version>/`.

### Notes

- The legacy Ant build (`build.xml`) delegates compression to `dev/build/build-bundle.xml`. The Gradle build is the primary supported path for this module and enforces the folder-in-archive layout directly in this repository.
- Packaging format is controlled by `bundle.format` in `build.properties` (`7z` or `zip`).
