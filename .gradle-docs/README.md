# Bearsampp Module Node.js - Gradle Build Documentation

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Build Tasks](#build-tasks)
- [Configuration](#configuration)
- [Architecture](#architecture)
- [Troubleshooting](#troubleshooting)
- [Additional Documentation](#additional-documentation)

---

## Overview

The Bearsampp Module Node.js project has been converted to a **pure Gradle build system**, replacing the legacy Ant build configuration. This provides:

- **Modern Build System**     - Native Gradle tasks and conventions
- **Better Performance**       - Incremental builds and caching
- **Simplified Maintenance**   - Pure Groovy/Gradle DSL
- **Enhanced Tooling**         - IDE integration and dependency management
- **Cross-Platform Support**   - Works on Windows, Linux, and macOS

### Project Information

| Property          | Value                                    |
|-------------------|------------------------------------------|
| **Project Name**  | module-nodejs                            |
| **Group**         | com.bearsampp.modules                    |
| **Type**          | Node.js Module Builder                   |
| **Build Tool**    | Gradle 8.x+                              |
| **Language**      | Groovy (Gradle DSL)                      |

---

## Quick Start

### Prerequisites

| Requirement       | Version       | Purpose                                  |
|-------------------|---------------|------------------------------------------|
| **Java**          | 8+            | Required for Gradle execution            |
| **Gradle**        | 8.0+          | Build automation tool                    |
| **7-Zip**         | Latest        | Archive extraction and creation          |

### Basic Commands

```bash
# Display build information
gradle info

# List all available tasks
gradle tasks

# Verify build environment
gradle verify

# Build a release (interactive)
gradle release

# Build a specific version (non-interactive)
gradle release -PbundleVersion=24.6.0

# Clean build artifacts
gradle clean
```

---

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/bearsampp/module-nodejs.git
cd module-nodejs
```

### 2. Verify Environment

```bash
gradle verify
```

This will check:
- Java version (8+)
- Required files (build.properties)
- Directory structure (bin/)
- Build dependencies

### 3. List Available Versions

```bash
gradle listVersions
```

### 4. Build Your First Release

```bash
# Interactive mode (prompts for version)
gradle release

# Or specify version directly
gradle release -PbundleVersion=24.6.0
```

---

## Build Tasks

### Core Build Tasks

| Task                  | Description                                      | Example                                  |
|-----------------------|--------------------------------------------------|------------------------------------------|
| `release`             | Build and package release (interactive/non-interactive) | `gradle release -PbundleVersion=24.6.0` |
| `releaseAll`          | Build all available versions in bin/             | `gradle releaseAll`                      |
| `clean`               | Clean build artifacts and temporary files        | `gradle clean`                           |

### Verification Tasks

| Task                      | Description                                  | Example                                      |
|---------------------------|----------------------------------------------|----------------------------------------------|
| `verify`                  | Verify build environment and dependencies    | `gradle verify`                              |
| `validateProperties`      | Validate build.properties configuration      | `gradle validateProperties`                  |

### Information Tasks

| Task                | Description                                      | Example                    |
|---------------------|--------------------------------------------------|----------------------------|
| `info`              | Display build configuration information          | `gradle info`              |
| `listVersions`      | List available bundle versions in bin/           | `gradle listVersions`      |
| `listReleases`      | List all available releases from modules-untouched | `gradle listReleases`    |
| `checkModulesUntouched` | Check modules-untouched integration          | `gradle checkModulesUntouched` |

### Task Groups

| Group            | Purpose                                          |
|------------------|--------------------------------------------------|
| **build**        | Build and package tasks                          |
| **verification** | Verification and validation tasks                |
| **help**         | Help and information tasks                       |

---

## Configuration

### build.properties

The main configuration file for the build:

```properties
bundle.name     = nodejs
bundle.release  = 2025.8.21
bundle.type     = bins
bundle.format   = 7z
```

| Property          | Description                          | Example Value  |
|-------------------|--------------------------------------|----------------|
| `bundle.name`     | Name of the bundle                   | `nodejs`       |
| `bundle.release`  | Release version                      | `2025.8.21`    |
| `bundle.type`     | Type of bundle                       | `bins`         |
| `bundle.format`   | Archive format                       | `7z`           |

### gradle.properties

Gradle-specific configuration:

```properties
# Gradle daemon configuration
org.gradle.daemon=true
org.gradle.parallel=true
org.gradle.caching=true

# JVM settings
org.gradle.jvmargs=-Xmx2g -XX:MaxMetaspaceSize=512m
```

### Directory Structure

```
module-nodejs/
├── .gradle-docs/          # Gradle documentation
│   ├── README.md          # Main documentation
│   └── PACKAGING.md       # Packaging guide
├── bin/                   # Node.js version bundles
│   ├── nodejs24.6.0/
│   ├── nodejs22.11.0/
│   ├── archived/          # Archived versions
│   └── ...
├── img/                   # Images and assets
│   └── Bearsampp-logo.svg
├── build.gradle           # Main Gradle build script
├── settings.gradle        # Gradle settings
├── build.properties       # Build configuration
├── releases.properties    # Available Node.js releases
└── README.md              # Main project README

bearsampp-build/                    # External build directory (outside repo)
├── tmp/                            # Temporary build files
│   ├── bundles_prep/bins/nodejs/   # Prepared bundles
│   ├── bundles_build/bins/nodejs/  # Build staging
│   ├── downloads/nodejs/           # Downloaded dependencies
│   └── extract/nodejs/             # Extracted archives
└── bins/nodejs/                    # Final packaged archives
    └── 2025.8.21/                  # Release version
        ├── bearsampp-nodejs-24.6.0-2025.8.21.7z
        ├── bearsampp-nodejs-24.6.0-2025.8.21.7z.md5
        └── ...
```

---

## Architecture

### Build Process Flow

```
1. User runs: gradle release -PbundleVersion=24.6.0
                    ↓
2. Validate environment and version
                    ↓
3. Check if Node.js binaries exist in bin/nodejs24.6.0/
                    ↓
4. If not found, download from modules-untouched
   - Check nodejs.properties for version URL
   - Fallback to standard URL format
   - Extract to tmp/extract/
                    ↓
5. Create preparation directory (tmp/bundles_prep/)
                    ↓
6. Copy Node.js files to prep directory
                    ↓
7. Overlay any custom files from bin/nodejs24.6.0/
                    ↓
8. Copy to bundles_build directory
                    ↓
9. Package prepared folder into archive
   - Archive includes top-level folder: nodejs24.6.0/
   - Generate hash files (MD5, SHA1, SHA256, SHA512)
                    ↓
10. Output to bearsampp-build/bins/nodejs/{bundle.release}/
```

### Packaging Details

- **Archive name format**: `bearsampp-nodejs-{version}-{bundle.release}.{7z|zip}`
- **Location**: `bearsampp-build/bins/nodejs/{bundle.release}/`
  - Example: `bearsampp-build/bins/nodejs/2025.8.21/bearsampp-nodejs-24.6.0-2025.8.21.7z`
- **Content root**: The top-level folder inside the archive is `nodejs{version}/` (e.g., `nodejs24.6.0/`)
- **Structure**: The archive contains the Node.js version folder at the root with all files inside

**Archive Structure Example**:
```
bearsampp-nodejs-24.6.0-2025.8.21.7z
└── nodejs24.6.0/           ← Version folder at root
    ├── node.exe
    ├── npm
    ├── npm.cmd
    ├── npx
    ├── npx.cmd
    ├── node_modules/
    └── ...
```

**Verification Commands**:

```bash
# List archive contents with 7z
7z l bearsampp-build/bins/nodejs/2025.8.21/bearsampp-nodejs-24.6.0-2025.8.21.7z | more

# You should see entries beginning with:
#   nodejs24.6.0/node.exe
#   nodejs24.6.0/npm
#   nodejs24.6.0/node_modules/
#   nodejs24.6.0/...

# Extract and inspect with PowerShell (zip example)
Expand-Archive -Path bearsampp-build/bins/nodejs/2025.8.21/bearsampp-nodejs-24.6.0-2025.8.21.zip -DestinationPath .\_inspect
Get-ChildItem .\_inspect\nodejs24.6.0 | Select-Object Name

# Expected output:
#   node.exe
#   npm
#   npm.cmd
#   node_modules/
#   ...
```

**Note**: This archive structure matches other Bearsampp modules (PHP, MySQL) where archives contain `{module}{version}/` at the root. This ensures consistency across all Bearsampp modules.

**Hash Files**: Each archive is accompanied by hash sidecar files:
- `.md5` - MD5 checksum
- `.sha1` - SHA-1 checksum
- `.sha256` - SHA-256 checksum
- `.sha512` - SHA-512 checksum

Example:
```
bearsampp-build/bins/nodejs/2025.8.21/
├── bearsampp-nodejs-24.6.0-2025.8.21.7z
├── bearsampp-nodejs-24.6.0-2025.8.21.7z.md5
├── bearsampp-nodejs-24.6.0-2025.8.21.7z.sha1
├── bearsampp-nodejs-24.6.0-2025.8.21.7z.sha256
└── bearsampp-nodejs-24.6.0-2025.8.21.7z.sha512
```

### Version Resolution Strategy

The build system uses a two-tier strategy to locate Node.js binaries:

1. **modules-untouched nodejs.properties** (Primary)
   - URL: `https://github.com/Bearsampp/modules-untouched/blob/main/modules/nodejs.properties`
   - Contains version-to-URL mappings
   - Example: `24.6.0=https://github.com/Bearsampp/modules-untouched/releases/download/nodejs-24.6.0/nodejs-24.6.0-win-x64.7z`

2. **Standard URL Format** (Fallback)
   - Format: `https://github.com/Bearsampp/modules-untouched/releases/download/nodejs-{version}/nodejs-{version}-win-x64.7z`
   - Used when version not found in nodejs.properties
   - Assumes standard naming convention

---

## Troubleshooting

### Common Issues

#### Issue: "Dev path not found"

**Symptom:**
```
Dev path not found: E:/Bearsampp-development/dev
```

**Solution:**
This is a warning only. The dev path is optional for most tasks. If you need it, ensure the `dev` project exists in the parent directory.

---

#### Issue: "Bundle version not found"

**Symptom:**
```
Bundle version not found in bin/ or bin/archived/
```

**Solution:**
1. List available versions: `gradle listVersions`
2. Use an existing version: `gradle release -PbundleVersion=24.6.0`
3. Or the build will automatically download from modules-untouched

---

#### Issue: "Failed to download from modules-untouched"

**Symptom:**
```
Failed to download from modules-untouched: Connection refused
```

**Solution:**
1. Check internet connectivity
2. Verify version exists: `gradle listReleases`
3. Check modules-untouched repository is accessible
4. Try again later if repository is temporarily unavailable

---

#### Issue: "Java version too old"

**Symptom:**
```
Java 8+ required
```

**Solution:**
1. Check Java version: `java -version`
2. Install Java 8 or higher
3. Update JAVA_HOME environment variable

---

#### Issue: "7-Zip not found"

**Symptom:**
```
7-Zip not found. Please install 7-Zip or set 7Z_HOME environment variable.
```

**Solution:**
1. Install 7-Zip from https://www.7-zip.org/
2. Or set `7Z_HOME` environment variable to 7-Zip installation directory
3. Or change `bundle.format` to `zip` in build.properties

---

### Debug Mode

Run Gradle with debug output:

```bash
gradle release -PbundleVersion=24.6.0 --info
gradle release -PbundleVersion=24.6.0 --debug
```

### Clean Build

If you encounter issues, try a clean build:

```bash
gradle clean
gradle release -PbundleVersion=24.6.0
```

---

## Migration Guide

### From Ant to Gradle

The project has been fully migrated from Ant to Gradle. Here's what changed:

#### Removed Files

| File              | Status    | Replacement                |
|-------------------|-----------|----------------------------|
| `build.xml`       | ✗ Removed | `build.gradle`             |

#### Command Mapping

| Ant Command                          | Gradle Command                              |
|--------------------------------------|---------------------------------------------|
| `ant release`                        | `gradle release`                            |
| `ant release -Dinput.bundle=24.6.0`  | `gradle release -PbundleVersion=24.6.0`     |
| `ant clean`                          | `gradle clean`                              |

#### Key Differences

| Aspect              | Ant                          | Gradle                           |
|---------------------|------------------------------|----------------------------------|
| **Build File**      | XML (build.xml)              | Groovy DSL (build.gradle)        |
| **Task Definition** | `<target name="...">`        | `tasks.register('...')`          |
| **Properties**      | `<property name="..." />`    | `ext { ... }`                    |
| **Dependencies**    | Manual downloads             | Automatic with repositories      |
| **Caching**         | None                         | Built-in incremental builds      |
| **IDE Support**     | Limited                      | Excellent (IntelliJ, Eclipse)    |

---

## Additional Documentation

This documentation is part of a comprehensive guide for the Bearsampp Node.js module build system:

### Documentation Files

| Document                  | Description                                      |
|---------------------------|--------------------------------------------------|
| [README.md](README.md)    | Main build documentation (this file)             |
| [PACKAGING.md](PACKAGING.md) | Packaging and archive structure guide         |
| [MIGRATION.md](MIGRATION.md) | Migration guide from Ant to Gradle            |

### Quick Links

- **Main Project README**: [../README.md](../README.md)
- **Build Script**: [../build.gradle](../build.gradle)
- **Build Configuration**: [../build.properties](../build.properties)
- **Gradle Settings**: [../settings.gradle](../settings.gradle)

### External Resources

- [Gradle Documentation](https://docs.gradle.org/)
- [Bearsampp Project](https://github.com/bearsampp/bearsampp)
- [Node.js Downloads](https://nodejs.org/en/download/)
- [modules-untouched Repository](https://github.com/Bearsampp/modules-untouched)

---

## Support

For issues and questions:

- **GitHub Issues**: https://github.com/bearsampp/module-nodejs/issues
- **Bearsampp Issues**: https://github.com/bearsampp/bearsampp/issues
- **Documentation**: https://bearsampp.com/module/nodejs

---

**Last Updated**: 2025-01-31  
**Version**: 2025.8.21  
**Build System**: Pure Gradle (no wrapper, no Ant)

### Notes

- This project deliberately does not ship the Gradle Wrapper. Install Gradle 8+ locally and run with `gradle ...`.
- Legacy Ant files have been removed. The project now uses pure Gradle for all build operations.
- See [MIGRATION.md](MIGRATION.md) for details on the Ant to Gradle migration.
