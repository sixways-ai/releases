# SixWays Release Feed

Public release feed for SixWays endpoint software. Customer instances poll this feed to discover available updates.

## Feed URLs

| URL | Description |
|-----|-------------|
| `/latest.json` | Latest stable release with full detail |
| `/index.json` | Complete release catalog |
| `/channels/stable.json` | Stable channel releases |
| `/channels/beta.json` | Beta channel releases |
| `/versions/{version}.json` | Per-version detail with artifacts |

Base URL: `https://sixways-ai.github.io/releases`

## Publishing a Release

1. Create a GitHub Release with a tag like `v1.0.0` (or `v1.1.0-beta.1` for pre-release)
2. Attach binary artifacts (pkg, deb, rpm, msi, tar.gz)
3. Optionally include YAML frontmatter in the release body for metadata:

```markdown
---
urgency: critical
min_version: 0.9.0
containers: base=ghcr.io/sixways-ai/sixways-sandbox:1.0.0-base@sha256:abc123
---

## What's New

- Feature X
- Bug fix Y
```

The GitHub Actions workflow automatically regenerates the JSON feed files when a release is published.

## Asset Naming Convention

The workflow parses platform, architecture, and artifact type from filenames:

| Pattern | Platform | Arch | Type |
|---------|----------|------|------|
| `*.pkg` | macos | universal | installer |
| `*macos*.tar.gz` | macos | x86_64/aarch64 | binary |
| `*.msi`, `*.exe` | windows | x86_64/aarch64 | installer |
| `*.deb` | linux | x86_64/aarch64 | installer_deb |
| `*.rpm` | linux | x86_64/aarch64 | installer_rpm |
| `*linux*.tar.gz` | linux | x86_64/aarch64 | binary |

Include `arm64` or `aarch64` in the filename for ARM builds.

## Schema Version

Current schema version: **1**

All JSON files include a `schema_version` field. Customer instances should check this field and handle unknown versions gracefully.

## Feed Signing

Feed JSON files are signed with Ed25519 (signify-openbsd) when `FEED_SIGNING_KEY` GitHub Actions secret is configured. Signature files are committed alongside feed JSON. Clients should verify signatures against the public key.

## GitHub Actions Secrets

| Secret | Purpose |
|--------|---------|
| `FEED_SIGNING_KEY` | Ed25519 private key (signify-openbsd format) used to sign feed JSON files. When present, the publish workflow generates `.sig` files alongside each feed JSON. |
