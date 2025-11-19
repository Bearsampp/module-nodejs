# Migration Guide: Ant to Gradle

This document provides guidance for migrating from the legacy Ant build system to the new Gradle build system.

## Overview

The Bearsampp Module Node.js project has been fully migrated from Apache Ant to Gradle. The Gradle build system provides better performance, modern tooling, and improved maintainability.

## What Changed

### Removed Files

The following files have been **removed** from the repository:

| File                          | Status    | Notes                                    |
|-------------------------------|-----------|------------------------------------------|
| `build.xml`                   | ✗ Removed | Replaced by `build.gradle`               |
| `module-nodejs.RELEASE.launch`| ✗ Removed | Eclipse Ant launcher, no longer needed   |

**Note**: These legacy Ant build files have been completely removed. The project now uses pure Gradle for all build operations.

### New Files

| File                          | Purpose                                      |
|-------------------------------|----------------------------------------------|
| `build.gradle`                | Main Gradle build script                     |
| `settings.gradle`             | Gradle project settings                      |
| `gradle.properties`           | Gradle configuration properties              |
| `.gradle-docs/`               | Gradle build documentation directory         |
| `.gradle-docs/README.md`      | Main build documentation                     |
| `.gradle-docs/PACKAGING.md`   | Packaging and archive structure guide        |
| `.gradle-docs/MIGRATION.md`   | This migration guide                         |

## Command Mapping

### Basic Commands

| Ant Command                          | Gradle Command                              | Notes                                    |
|--------------------------------------|---------------------------------------------|------------------------------------------|
| `ant release`                        | `gradle release`                            | Interactive mode (prompts for version)   |
| `ant release -Dinput.bundle=24.6.0`  | `gradle release -PbundleVersion=24.6.0`     | Non-interactive mode                     |
| `ant clean`                          | `gradle clean`                              | Clean build artifacts                    |

### New Commands

Gradle provides additional commands not available in Ant:

| Command                       | Description                                  |
|-------------------------------|----------------------------------------------|
| `gradle info`                 | Display build configuration information      |
| `gradle tasks`                | List all available tasks                     |
| `gradle verify`               | Verify build environment and dependencies    |
| `gradle listVersions`         | List available versions in bin/              |
| `gradle listReleases`         | List releases from modules-untouched         |
| `gradle releaseAll`           | Build all versions in bin/                   |
| `gradle validateProperties`   | Validate build.properties configuration      |
| `gradle checkModulesUntouched`| Check modules-untouched integration          |

## Key Differences

### 1. Build File Format

**Ant (build.xml):**
```xml
<project name="module-nodejs" basedir=".">
  <target name="release.build">
    <copy todir="${bundle.tmp.prep.path}/${bundle.folder}">
      <fileset dir="${nodeextract.path}"/>
    </copy>
  </target>
</project>
```

**Gradle (build.gradle):**
```groovy
tasks.register('release') {
    group = 'build'
    description = 'Build release package'
    
    doLast {
        copy {
            from nodeExtractPath
            into bundlePrepPath
        }
    }
}
```

### 2. Property Handling

**Ant:**
```xml
<property name="bundle.name" value="nodejs"/>
<property file="build.properties"/>
```

**Gradle:**
```groovy
def buildProps = new Properties()
file('build.properties').withInputStream { buildProps.load(it) }

ext {
    bundleName = buildProps.getProperty('bundle.name', 'nodejs')
}
```

### 3. Task Execution

**Ant:**
- Tasks defined as `<target>` elements
- Dependencies specified with `depends` attribute
- Sequential execution by default

**Gradle:**
- Tasks defined with `tasks.register()`
- Dependencies specified with `dependsOn`
- Parallel execution possible
- Incremental builds supported

### 4. Dependency Management

**Ant:**
- Manual download and extraction
- No built-in dependency management
- Custom scripts required

**Gradle:**
- Built-in dependency management
- Automatic download and caching
- Repository support (Maven, etc.)

### 5. IDE Integration

**Ant:**
- Limited IDE support
- Manual configuration required
- Eclipse `.launch` files needed

**Gradle:**
- Excellent IDE support (IntelliJ IDEA, Eclipse, VS Code)
- Automatic project import
- No manual configuration needed

## Migration Steps

If you're migrating from Ant to Gradle in your own project:

### Step 1: Install Gradle

```bash
# Windows (Chocolatey)
choco install gradle

# macOS (Homebrew)
brew install gradle

# Linux (SDKMAN!)
sdk install gradle

# Verify installation
gradle --version
```

### Step 2: Create build.gradle

Create a new `build.gradle` file based on the current implementation. See [build.gradle](../build.gradle) for reference.

### Step 3: Create settings.gradle

```groovy
rootProject.name = 'module-nodejs'
```

### Step 4: Test the Build

```bash
# Verify environment
gradle verify

# List available tasks
gradle tasks

# Test a build
gradle release -PbundleVersion=24.6.0
```

### Step 5: Update Documentation

Update your README.md and other documentation to reference Gradle commands instead of Ant.

### Step 6: Deprecate Ant Files

Mark `build.xml` and related files as deprecated. You can keep them for reference but note they're no longer used.

## Troubleshooting Migration Issues

### Issue: "Cannot find dev project"

**Ant Behavior:**
```xml
<property name="dev.path" location="${root.dir}/dev"/>
<fail unless="dev.path" message="Project 'dev' not found"/>
```

**Gradle Behavior:**
```groovy
ext {
    devPath = file("${rootDir}/dev").absolutePath
}

if (!file(ext.devPath).exists()) {
    throw new GradleException("Dev path not found: ${ext.devPath}")
}
```

**Solution:**
Ensure the `dev` project exists in the parent directory, or modify the build script if your structure is different.

### Issue: "Property not found"

**Ant Behavior:**
Properties loaded from `build.properties` are automatically available.

**Gradle Behavior:**
Properties must be explicitly loaded:

```groovy
def buildProps = new Properties()
file('build.properties').withInputStream { buildProps.load(it) }
```

**Solution:**
Check that `build.properties` exists and properties are loaded correctly.

### Issue: "Task not found"

**Ant Behavior:**
```bash
ant release
```

**Gradle Behavior:**
```bash
gradle release
```

**Solution:**
Use `gradle tasks` to list all available tasks and verify the task name.

## Benefits of Gradle

### 1. Performance

- **Incremental builds**: Only rebuild what changed
- **Build cache**: Reuse outputs from previous builds
- **Parallel execution**: Run tasks in parallel when possible

### 2. Maintainability

- **Groovy DSL**: More readable and maintainable than XML
- **Modular**: Easy to split build logic into multiple files
- **Reusable**: Share build logic across projects

### 3. Tooling

- **IDE integration**: Excellent support in IntelliJ IDEA, Eclipse, VS Code
- **Build scans**: Detailed build performance analysis
- **Dependency insight**: Understand dependency trees

### 4. Ecosystem

- **Plugins**: Thousands of plugins available
- **Community**: Large and active community
- **Documentation**: Comprehensive official documentation

## Best Practices

### 1. Use Gradle Wrapper (Optional)

While this project doesn't include the Gradle Wrapper, you can add it:

```bash
gradle wrapper --gradle-version 8.5
```

This creates:
- `gradlew` (Unix)
- `gradlew.bat` (Windows)
- `gradle/wrapper/` directory

Benefits:
- Ensures consistent Gradle version across team
- No need to install Gradle separately

### 2. Organize Build Logic

Keep build logic organized:
- Main build logic in `build.gradle`
- Configuration in `build.properties`
- Documentation in `.gradle-docs/`

### 3. Use Task Groups

Organize tasks into logical groups:

```groovy
tasks.register('myTask') {
    group = 'build'  // or 'verification', 'help', etc.
    description = 'My task description'
}
```

### 4. Leverage Gradle Features

- Use `ext` for shared properties
- Use `doFirst` and `doLast` for task actions
- Use `dependsOn` for task dependencies
- Use `mustRunAfter` for task ordering

## Additional Resources

### Official Documentation

- [Gradle User Manual](https://docs.gradle.org/current/userguide/userguide.html)
- [Gradle DSL Reference](https://docs.gradle.org/current/dsl/)
- [Gradle Build Language Reference](https://docs.gradle.org/current/userguide/writing_build_scripts.html)

### Tutorials

- [Getting Started with Gradle](https://docs.gradle.org/current/userguide/getting_started.html)
- [Migrating from Ant to Gradle](https://docs.gradle.org/current/userguide/migrating_from_ant.html)
- [Gradle Best Practices](https://docs.gradle.org/current/userguide/authoring_maintainable_build_scripts.html)

### Bearsampp Resources

- [Bearsampp Project](https://github.com/bearsampp/bearsampp)
- [Module PHP (Gradle)](https://github.com/Bearsampp/module-php/tree/gradle-convert)
- [modules-untouched Repository](https://github.com/Bearsampp/modules-untouched)

## Support

If you encounter issues during migration:

1. Check this migration guide
2. Review the [main documentation](README.md)
3. Check [troubleshooting guide](README.md#troubleshooting)
4. Open an issue on [GitHub](https://github.com/bearsampp/module-nodejs/issues)

---

**Last Updated**: 2025-01-31  
**Version**: 2025.8.21  
**Build System**: Pure Gradle

## Summary

The migration from Ant to Gradle provides:

✅ **Better Performance** - Incremental builds and caching  
✅ **Modern Tooling** - Excellent IDE integration  
✅ **Easier Maintenance** - Readable Groovy DSL  
✅ **More Features** - Rich plugin ecosystem  
✅ **Better Documentation** - Comprehensive guides  

The Gradle build system is now the primary and recommended way to build Bearsampp Node.js modules.
