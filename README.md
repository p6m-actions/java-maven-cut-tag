# Java Maven Cut Tag

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/java-maven-cut-tag?style=flat-square&label=Latest%20Release&color=blue)

## Description

A GitHub Action for cutting versioned tags for Java Maven projects. This action automates the process of bumping version numbers in `pom.xml` files, creating Git tags in the `vMajor.Minor.Patch` format, and pushing them to your repository.

## Usage

This action requires write permissions to your repository to create tags and push changes. Make sure to include `contents: write` in your workflow permissions.

```yaml
- name: Cut Tag
  uses: p6m-actions/java-maven-cut-tag@v1
  with:
    version-level: "patch"
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `version-level` | Version bump level. Options: `patch`, `minor`, `major` | No | `patch` |

## Outputs

| Name | Description |
|------|-------------|
| `version` | The new version number (e.g., `1.2.3`) |
| `tag` | The newly created tag (e.g., `v1.2.3`) |

## Examples

### Basic Usage

```yaml
name: Build and Release

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'

      - name: Build with Maven
        run: mvn clean package

      - name: Cut Tag
        uses: p6m-actions/java-maven-cut-tag@v1
        with:
          version-level: "patch"
```

### Version Bumping Scenarios

```yaml
# Patch version bump (1.0.0 -> 1.0.1)
- name: Cut Patch Release
  uses: p6m-actions/java-maven-cut-tag@v1
  with:
    version-level: "patch"

# Minor version bump (1.0.0 -> 1.1.0)
- name: Cut Minor Release
  uses: p6m-actions/java-maven-cut-tag@v1
  with:
    version-level: "minor"

# Major version bump (1.0.0 -> 2.0.0)
- name: Cut Major Release
  uses: p6m-actions/java-maven-cut-tag@v1
  with:
    version-level: "major"
```

### Using Outputs

```yaml
- name: Cut Tag
  id: cut-tag
  uses: p6m-actions/java-maven-cut-tag@v1
  with:
    version-level: "minor"

- name: Create Release
  uses: actions/create-release@v1
  with:
    tag_name: ${{ steps.cut-tag.outputs.tag }}
    release_name: Release ${{ steps.cut-tag.outputs.version }}
```

## Requirements

- Java Maven project with a `pom.xml` file
- Repository must be checked out with full git history
- Write permissions to the repository (`contents: write`)
- Java and Maven must be available (the action sets up Java 21 by default)

## Behavior

1. **Validates input**: Ensures `version-level` is one of `patch`, `minor`, or `major`
2. **Checks repository state**: Verifies no uncommitted changes exist
3. **Reads current version**: Extracts version from the root `pom.xml`
4. **Calculates new version**: Increments based on the specified level
5. **Updates Maven files**: Uses Maven versions plugin to update all `pom.xml` files
6. **Creates commit**: Commits the version changes
7. **Creates and pushes tag**: Creates a Git tag and pushes both commit and tag

## Multi-module Projects

This action supports Maven multi-module projects and will automatically update version numbers in all child module `pom.xml` files.