<p align="center"><a href="https://bearsampp.com/contribute" target="_blank"><img width="250" src="img/Bearsampp-logo.svg"></a></p>

[![GitHub release](https://img.shields.io/github/release/bearsampp/module-nodejs.svg?style=flat-square)](https://github.com/bearsampp/module-nodejs/releases/latest)
![Total downloads](https://img.shields.io/github/downloads/bearsampp/module-nodejs/total.svg?style=flat-square)

This is a module of [Bearsampp project](https://github.com/bearsampp/bearsampp) involving Node.js.

## Build (Gradle)

This module now uses a pure Gradle build similar to the Bruno module.

Quick commands:

- List tasks: `gradle tasks`
- Show build info: `gradle info`
- Verify environment: `gradle verify`
- List local versions (bin and bin/archived): `gradle listVersions`
- List releases from modules-untouched: `gradle listReleases`
- Build a specific version: `gradle release -PbundleVersion=24.6.0`
- Build interactively (choose from local versions): `gradle release`
- Build all local versions (prep/copy flow): `gradle releaseAll`
- Clean: `gradle clean`

Archive layout assurance:

- The packaged archive includes the top-level version folder (e.g., `nodejs24.6.0/`).
- Example: `bearsampp-nodejs-24.6.0-2025.8.21.7z` contains `nodejs24.6.0/` as the root, with files inside it.

Verify quickly after a build:

- 7z: `7z l bearsampp-nodejs-<ver>-<release>.7z | findstr nodejs<ver>` should list the folder.
- Zip: Inspect contents; the first entry should be `nodejs<ver>/`.

Version resolution strategy:

1. Remote `modules-untouched` `nodejs.properties`
   - URL: `https://github.com/Bearsampp/modules-untouched/blob/main/modules/nodejs.properties`
2. Fallback constructed URL: `.../releases/download/nodejs-{version}/nodejs-{version}-win-x64.7z`

Output locations and overrides:

- Default base output: `<repo-root>/../bearsampp-build`
- Override base output via either:
  - `build.properties` property `build.path`
  - Environment variable `BEARSAMPP_BUILD_PATH`

Packaging settings come from `build.properties`:

- `bundle.name = nodejs`
- `bundle.type = bins`
- `bundle.format = 7z` (requires 7‑Zip)

7‑Zip detection (Windows):

- Auto-detects from `7Z_HOME` or common install paths, falls back to `where 7z.exe`.

Documentation index: see `/.gradle-docs/README.md`.

## Documentation and downloads

https://bearsampp.com/module/nodejs

Additional Gradle build docs for this module live in `/.gradle-docs/` of this repository.

## Issues

Issues must be reported on [Bearsampp repository](https://github.com/bearsampp/bearsampp/issues).
