# SixWays Release Feed

Public release feed for all SixWays components. Customer instances and tooling poll this feed to discover available updates.

## Components

| Component | Type | Artifacts |
|-----------|------|-----------|
| `endpoint` | Binary | pkg, deb, rpm, msi, tar.gz |
| `server` | Docker | Docker image + digest |
| `gateway` | Docker | Docker image + digest |
| `ml-gateway` | Docker | Docker image + digest |
| `sandbox` | Docker | Docker image + digest |
| `vscode-extension` | VSIX | VS Code extension package |

## Feed URLs

| URL | Description |
|-----|-------------|
| `/latest.json` | Latest release per component |
| `/index.json` | Complete release catalog (all components) |
| `/channels/stable.json` | Stable channel releases |
| `/channels/beta.json` | Beta channel releases |
| `/components/{component}/latest.json` | Per-component latest stable/beta |
| `/versions/{component}/{version}.json` | Per-version detail with artifacts |

Base URL: `https://sixways-ai.github.io/releases`

## Tag Format

Releases use component-prefixed tags:

```
endpoint/v1.3.0
server/v1.3.0
gateway/v2.0.0-beta.1
ml-gateway/v1.0.0
sandbox/v1.2.0
vscode-extension/v0.5.0
```

Legacy tags without a prefix (e.g., `v1.0.0`) are treated as `endpoint` releases for backwards compatibility.

## Publishing a Release

### Endpoint Release (binary assets)

1. Create a GitHub Release with tag `endpoint/v1.3.0`
2. Attach binary artifacts (pkg, deb, rpm, msi, tar.gz)
3. Include YAML frontmatter in the release body:

```markdown
---
urgency: recommended
min_version: 1.2.0
sha256_sixways_1_3_0_macos_aarch64_tar_gz: abc123...
---

## What's New

- Improved detection engine
- Fixed false positive in network monitor
```

### Server Release (Docker image)

1. Create a GitHub Release with tag `server/v1.3.0`
2. Include Docker image details in YAML frontmatter:

```markdown
---
urgency: recommended
docker_image: ghcr.io/sixways-ai/sixways-server:1.3.0
docker_digest: sha256:abc123def456...
helm_chart_version: 1.3.0
---

## What's New

- Added multi-tenant policy support
- Performance improvements for large fleets
```

### VS Code Extension Release

1. Create a GitHub Release with tag `vscode-extension/v0.5.0`
2. Include VSIX URL in frontmatter:

```markdown
---
urgency: recommended
vsix_url: https://github.com/sixways-ai/vscode-extension/releases/download/v0.5.0/sixways-0.5.0.vsix
---

## What's New

- Inline threat annotations
```

## Frontmatter Keys

| Key | Description |
|-----|-------------|
| `urgency` | Update urgency: `critical`, `recommended`, `optional` |
| `min_version` | Minimum version required before upgrading |
| `component` | Component name (optional, inferred from tag) |
| `docker_image` | Full Docker image reference (for Docker components) |
| `docker_digest` | Docker image digest (for Docker components) |
| `vsix_url` | Download URL for VSIX package |
| `helm_chart_version` | Associated Helm chart version |
| `containers` | Legacy container format (deprecated) |
| `sha256_*` | SHA-256 hash for a specific binary asset |

## Asset Naming Convention (endpoint)

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

Current schema version: **2**

All JSON files include a `schema_version` field. Consumers should check this field and handle unknown versions gracefully.

## Feed Signing

Feed JSON files are signed with Ed25519 (signify-openbsd) when `FEED_SIGNING_KEY` GitHub Actions secret is configured. Signature files are committed alongside feed JSON, including per-component files. Clients should verify signatures against the public key.

## Signature Verification

The Ed25519 public key used to sign the feed is distributed in the [`keys/`](keys/) directory. To verify a feed file:

```bash
signify-openbsd -V -p keys/feed.pub -m index.json -x index.json.sig
```

All feed JSON files have corresponding `.sig` files when signing is enabled. See [`keys/README.md`](keys/README.md) for details on keypair generation and key management.

## GitHub Actions Secrets

| Secret | Purpose |
|--------|---------|
| `FEED_SIGNING_KEY` | Ed25519 private key (signify-openbsd format) used to sign feed JSON files. When present, the publish workflow generates `.sig` files alongside each feed JSON. |
