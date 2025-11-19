# Packaging and Archive Layout

This document describes how the Bearsampp Node.js module packages releases and the structure of the resulting archives.

## Overview

The Gradle build produces archives that include the top-level version folder, matching the behavior used across all Bearsampp modules (PHP, MySQL, etc.). This ensures consistency and proper extraction behavior.

## Archive Structure

### Example for Node.js 24.6.0

```
bearsampp-nodejs-24.6.0-2025.8.21.7z
└── nodejs24.6.0/              ← Version folder at root
    ├── node.exe               ← Node.js executable
    ├── npm                    ← NPM script
    ├── npm.cmd                ← NPM Windows command
    ├── npx                    ← NPX script
    ├── npx.cmd                ← NPX Windows command
    ├── node_modules/          ← Node modules directory
    │   ├── npm/
    │   └── ...
    ├── LICENSE
    ├── README.md
    └── ...
```

### Key Points

1. **Top-level folder**: The archive contains `nodejs{version}/` as the root directory
2. **Version naming**: Format is `nodejs` + version number (e.g., `nodejs24.6.0`)
3. **Complete structure**: All Node.js files and directories are inside the version folder
4. **Consistency**: Matches the pattern used by other Bearsampp modules

## Implementation Details

### Compression Process

The build system uses two different approaches depending on the archive format:

#### 7-Zip Compression

```groovy
// Run from parent directory and include folder name explicitly
def parentDir = prepPath.parentFile
def folderName = prepPath.name  // e.g., "nodejs24.6.0"

ProcessBuilder([
    sevenZipExe,
    'a',                        // Add to archive
    '-t7z',                     // 7z format
    archiveFile.absolutePath,   // Output archive
    folderName                  // Folder to include
])
.directory(parentDir)           // Run from parent directory
.start()
```

**Why this works:**
- Running from the parent directory ensures the folder name is included in the archive
- Passing the folder name (not path) as the argument includes it at the root level
- Result: Archive contains `nodejs24.6.0/` at the root

#### ZIP Compression

```groovy
task("zipArchive_${bundleVersion}", type: Zip) {
    from(prepPath.parentFile) {
        include "${prepPath.name}/**"
    }
    destinationDirectory = archiveFile.parentFile
    archiveFileName = archiveFile.name
}
```

**Why this works:**
- `from(prepPath.parentFile)` starts from the parent directory
- `include "${prepPath.name}/**"` includes the folder and all its contents
- Result: Archive contains `nodejs24.6.0/` at the root

## Archive Naming Convention

Archives follow this naming pattern:

```
bearsampp-{bundle.name}-{version}-{bundle.release}.{format}
```

**Example:**
```
bearsampp-nodejs-24.6.0-2025.8.21.7z
```

**Components:**
- `bearsampp` - Project prefix
- `nodejs` - Bundle name (from build.properties)
- `24.6.0` - Node.js version
- `2025.8.21` - Bundle release date (from build.properties)
- `7z` - Archive format (from build.properties)

## Hash Files

Each archive is accompanied by multiple hash files for integrity verification:

```
bearsampp-nodejs-24.6.0-2025.8.21.7z
bearsampp-nodejs-24.6.0-2025.8.21.7z.md5      ← MD5 checksum
bearsampp-nodejs-24.6.0-2025.8.21.7z.sha1     ← SHA-1 checksum
bearsampp-nodejs-24.6.0-2025.8.21.7z.sha256   ← SHA-256 checksum
bearsampp-nodejs-24.6.0-2025.8.21.7z.sha512   ← SHA-512 checksum
```

### Hash File Format

Each hash file contains:
```
{hash_value} {filename}
```

**Example (MD5):**
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6 bearsampp-nodejs-24.6.0-2025.8.21.7z
```

### Verification

To verify archive integrity:

```bash
# Windows (PowerShell)
# MD5
Get-FileHash bearsampp-nodejs-24.6.0-2025.8.21.7z -Algorithm MD5

# SHA256
Get-FileHash bearsampp-nodejs-24.6.0-2025.8.21.7z -Algorithm SHA256

# Linux/macOS
# MD5
md5sum bearsampp-nodejs-24.6.0-2025.8.21.7z

# SHA256
sha256sum bearsampp-nodejs-24.6.0-2025.8.21.7z
```

## Output Locations

### Default Structure

```
bearsampp-build/
├── tmp/                                    # Temporary build files
│   ├── bundles_prep/bins/nodejs/          # Prepared bundles
│   │   └── nodejs24.6.0/                  # Prepared version
│   ├── bundles_build/bins/nodejs/         # Build staging
│   │   └── nodejs24.6.0/                  # Built version (non-archived)
│   ├── downloads/nodejs/                  # Downloaded archives
│   │   └── nodejs-24.6.0-win-x64.7z
│   └── extract/nodejs/                    # Extracted archives
│       └── 24.6.0/
│           └── node-v24.6.0-win-x64/
└── bins/nodejs/                           # Final packaged archives
    └── 2025.8.21/                         # Release version
        ├── bearsampp-nodejs-24.6.0-2025.8.21.7z
        ├── bearsampp-nodejs-24.6.0-2025.8.21.7z.md5
        ├── bearsampp-nodejs-24.6.0-2025.8.21.7z.sha1
        ├── bearsampp-nodejs-24.6.0-2025.8.21.7z.sha256
        └── bearsampp-nodejs-24.6.0-2025.8.21.7z.sha512
```

### Customizing Output Location

You can override the default output location in two ways:

#### 1. build.properties

```properties
build.path = C:/Custom-Build-Path
```

#### 2. Environment Variable

```bash
# Windows (PowerShell)
$env:BEARSAMPP_BUILD_PATH = "C:\Custom-Build-Path"

# Windows (Command Prompt)
set BEARSAMPP_BUILD_PATH=C:\Custom-Build-Path

# Linux/macOS
export BEARSAMPP_BUILD_PATH=/path/to/custom/build
```

**Priority:**
1. `build.properties` `build.path` property (highest)
2. `BEARSAMPP_BUILD_PATH` environment variable
3. Default: `{repo-parent}/bearsampp-build` (lowest)

## Verification Guide

### How to Verify Archive Structure

After building a release, verify the archive structure:

#### Method 1: Using 7-Zip Command Line

```bash
# List archive contents
7z l bearsampp-nodejs-24.6.0-2025.8.21.7z

# Filter for the version folder
7z l bearsampp-nodejs-24.6.0-2025.8.21.7z | findstr nodejs24.6.0

# Expected output should show entries like:
#   nodejs24.6.0\node.exe
#   nodejs24.6.0\npm
#   nodejs24.6.0\node_modules\...
```

#### Method 2: Using PowerShell (ZIP)

```powershell
# Extract to temporary directory
Expand-Archive -Path bearsampp-nodejs-24.6.0-2025.8.21.zip -DestinationPath .\_inspect

# List contents
Get-ChildItem .\_inspect

# Expected output:
#   Directory: E:\...\\_inspect
#   Mode    LastWriteTime    Length Name
#   ----    -------------    ------ ----
#   d-----  1/31/2025  ...          nodejs24.6.0

# List files inside version folder
Get-ChildItem .\_inspect\nodejs24.6.0

# Clean up
Remove-Item .\_inspect -Recurse -Force
```

#### Method 3: Using 7-Zip GUI

1. Open the archive in 7-Zip
2. The first entry should be a folder named `nodejs{version}/`
3. All files should be inside this folder

### Quick Verification Script

```bash
# Build a release
gradle release -PbundleVersion=24.6.0

# Verify the archive
7z l bearsampp-build/bins/nodejs/2025.8.21/bearsampp-nodejs-24.6.0-2025.8.21.7z | findstr /C:"nodejs24.6.0"

# If successful, you should see multiple lines starting with "nodejs24.6.0\"
```

## Build Process Flow

### Step-by-Step Packaging

1. **Preparation Phase**
   ```
   tmp/bundles_prep/bins/nodejs/nodejs24.6.0/
   ```
   - Copy Node.js files from source (downloaded or local)
   - Overlay custom files from bin/nodejs24.6.0/
   - Result: Complete Node.js installation ready for packaging

2. **Build Phase**
   ```
   tmp/bundles_build/bins/nodejs/nodejs24.6.0/
   ```
   - Copy prepared files to build directory
   - This is the non-archived version
   - Can be used for testing before packaging

3. **Packaging Phase**
   ```
   bins/nodejs/2025.8.21/bearsampp-nodejs-24.6.0-2025.8.21.7z
   ```
   - Compress from parent directory
   - Include version folder at root
   - Generate hash files

## Troubleshooting

### Issue: Archive doesn't contain version folder

**Symptom:**
```
Archive contains files directly at root instead of nodejs24.6.0/ folder
```

**Solution:**
This should not happen with the current build system. If it does:
1. Check that you're using the latest build.gradle
2. Verify the compression command is running from the parent directory
3. Check the Gradle output for any errors during packaging

### Issue: Wrong folder name in archive

**Symptom:**
```
Archive contains "node-v24.6.0-win-x64" instead of "nodejs24.6.0"
```

**Solution:**
This indicates the preparation phase didn't complete correctly:
1. Check that the prep directory was created: `tmp/bundles_prep/bins/nodejs/nodejs24.6.0/`
2. Verify files were copied to the prep directory
3. Run with `--info` flag to see detailed output: `gradle release -PbundleVersion=24.6.0 --info`

### Issue: Hash files not generated

**Symptom:**
```
Archive created but no .md5, .sha1, etc. files
```

**Solution:**
1. Check Gradle output for hash generation errors
2. Verify Java has permissions to write to the output directory
3. Try running with `--stacktrace` to see detailed error: `gradle release -PbundleVersion=24.6.0 --stacktrace`

## Best Practices

### 1. Always Verify After Build

After building a release, always verify the archive structure:

```bash
gradle release -PbundleVersion=24.6.0
7z l bearsampp-build/bins/nodejs/2025.8.21/bearsampp-nodejs-24.6.0-2025.8.21.7z | findstr nodejs24.6.0
```

### 2. Test Extraction

Before distributing, test that the archive extracts correctly:

```bash
# Create test directory
mkdir test-extract
cd test-extract

# Extract archive
7z x ../bearsampp-build/bins/nodejs/2025.8.21/bearsampp-nodejs-24.6.0-2025.8.21.7z

# Verify structure
dir nodejs24.6.0

# Test Node.js
nodejs24.6.0\node.exe --version

# Clean up
cd ..
rmdir /s /q test-extract
```

### 3. Verify Hash Files

Always verify hash files are generated and correct:

```bash
# Check hash file exists
dir bearsampp-build\bins\nodejs\2025.8.21\*.md5

# Verify hash matches
Get-FileHash bearsampp-build/bins/nodejs/2025.8.21/bearsampp-nodejs-24.6.0-2025.8.21.7z -Algorithm MD5
type bearsampp-build\bins\nodejs\2025.8.21\bearsampp-nodejs-24.6.0-2025.8.21.7z.md5
```

## Related Documentation

- [Main Build Documentation](README.md) - Complete build system documentation
- [build.gradle](../build.gradle) - Build script implementation
- [build.properties](../build.properties) - Build configuration

---

**Last Updated**: 2025-01-31  
**Version**: 2025.8.21
