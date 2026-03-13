# SixWays Releases

Public release feed for SixWays endpoint software updates.

## Stack

- **Format**: Static JSON files served via GitHub Pages
- **CI**: GitHub Actions (auto-generates feeds on release publish)
- **Base URL**: `https://sixways-ai.github.io/releases`

## Structure

```
index.json              # Complete release catalog
latest.json             # Latest stable release
channels/
  stable.json           # Stable channel
  beta.json             # Beta channel
versions/
  {version}.json        # Per-version detail with artifacts
```

## How It Works

1. Create a GitHub Release with tag (e.g., `v1.0.0` or `v1.1.0-beta.1`)
2. Attach binary artifacts (pkg, deb, rpm, msi, tar.gz)
3. Optionally add YAML frontmatter in release body (urgency, min_version, containers)
4. GitHub Actions workflow regenerates all JSON feed files

## Schema

- Current schema version: 1
- All JSON files include `schema_version` field
- Asset naming parsed for platform/arch/type (e.g., `*.pkg` = macOS, `*.msi` = Windows)

## Consumed By

SixWays Server polls this feed to discover upstream releases and manage fleet updates.
