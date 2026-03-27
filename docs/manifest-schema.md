# Update Manifest Schema Reference

The `sixways-update-manifest.json` file ships with each GitHub Release and contains
Ed25519 signatures for all artifacts. Endpoints verify this manifest before installing
any update.

## Schema Version

Current: **2**

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `schema_version` | integer | Yes | Schema version (currently `2`) |
| `version` | string | Yes | Semver version (e.g., `"1.2.0"`) |
| `channel` | string | Yes | `"stable"`, `"beta"`, or `"canary"` (canary is planned -- not yet implemented) |
| `commit_sha` | string | No | Git commit SHA for provenance |
| `build_timestamp` | string | Yes | ISO 8601 UTC build time |
| `min_supported_version` | string | No | Minimum version required before upgrading |
| `release_notes_url` | string | No | URL to full release notes |
| `release_notes_summary` | string | No | Brief summary (max ~200 chars) |
| `urgency` | string | Yes | `"critical"`, `"recommended"`, or `"optional"` |
| `force` | boolean | Yes | Force update regardless of policy |
| `next_public_key` | string | No | Base64-encoded Ed25519 public key for key rotation |
| `revocation_url` | string | No | URL to the signed revocation list |
| `artifacts` | array | Yes | List of artifact entries (see below) |
| `manifest_signature` | string | Yes | Base64-encoded Ed25519 signature over canonical manifest |

### Artifact Entry

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `platform` | string | Yes | `"windows-x86_64"`, `"macos-universal"`, `"linux-x86_64"`, `"linux-aarch64"` |
| `format` | string | Yes | `"msi"`, `"msix"`, `"pkg"`, `"deb"`, `"rpm"`, `"zip"`, `"tar.gz"`, `"exe"` |
| `filename` | string | Yes | Artifact filename |
| `download_url` | string | Yes | Download URL (GitHub release asset or CDN) |
| `sha256` | string | Yes | Hex-encoded SHA-256 hash of the artifact |
| `size_bytes` | integer | Yes | File size in bytes |
| `signature` | string | Yes | Base64-encoded Ed25519 signature over the raw artifact bytes |

## Example

```json
{
  "schema_version": 2,
  "version": "1.2.0",
  "channel": "stable",
  "commit_sha": "a1b2c3d4e5f6...",
  "build_timestamp": "2026-03-26T10:00:00Z",
  "min_supported_version": "1.0.0",
  "release_notes_url": "https://github.com/sixways-ai/releases/releases/tag/endpoint/v1.2.0",
  "release_notes_summary": "Security improvements and bug fixes",
  "urgency": "recommended",
  "force": false,
  "next_public_key": null,
  "revocation_url": "https://sixways-ai.github.io/releases/revocations.json",
  "artifacts": [
    {
      "platform": "windows-x86_64",
      "format": "msi",
      "filename": "SixWays-Endpoint-1.2.0-x64.msi",
      "download_url": "https://github.com/sixways-ai/releases/releases/download/endpoint/v1.2.0/SixWays-Endpoint-1.2.0-x64.msi",
      "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
      "size_bytes": 15000000,
      "signature": "BASE64_ED25519_SIGNATURE..."
    },
    {
      "platform": "macos-universal",
      "format": "pkg",
      "filename": "SixWays-Endpoint-1.2.0-universal.pkg",
      "download_url": "https://github.com/sixways-ai/releases/releases/download/endpoint/v1.2.0/SixWays-Endpoint-1.2.0-universal.pkg",
      "sha256": "abc123def456...",
      "size_bytes": 18000000,
      "signature": "BASE64_ED25519_SIGNATURE..."
    }
  ],
  "manifest_signature": "BASE64_ED25519_SIGNATURE_OVER_CANONICAL_JSON..."
}
```

## Manifest Signature

The `manifest_signature` is computed over the **canonical JSON** of the manifest with
the `manifest_signature` field removed. To verify:

1. Parse the manifest JSON
2. Remove the `manifest_signature` field
3. Re-serialize to JSON (serde field order is the canonical order)
4. Verify the Ed25519 signature over those bytes against the pinned public key

## Manual Verification

```bash
# Using the endpoint binary's built-in verification
sixwaysd verify-manifest --manifest sixways-update-manifest.json

# Using the xtask tool (development)
cargo xtask verify-manifest --manifest sixways-update-manifest.json --key keys/update_pubkey.bin
```

## Two Signing Keys

| Key | Purpose | Location |
|-----|---------|----------|
| **Artifact signing key** | Signs artifacts + manifests | Offline (developer workstation or HSM) |
| **Feed signing key** | Signs the releases feed JSON files | `FEED_SIGNING_KEY` GitHub Actions secret |

The artifact signing key is the primary trust anchor. The feed signing key is
defense-in-depth. See [keys/README.md](../keys/README.md) for details.
