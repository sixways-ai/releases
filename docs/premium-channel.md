# Premium Channel

The premium channel delivers endpoint builds that include licensed premium features
such as Agent Intelligence. This document describes how premium builds are published,
discovered, and consumed.

## Overview

Premium builds are a superset of stable builds with additional feature modules
compiled in behind feature flags. They follow the same release feed schema but
include a `features` array that declares which premium capabilities are bundled.

## Tag Convention

Premium releases use a `-premium.N` prerelease suffix:

```
endpoint/v1.3.0-premium.1
endpoint/v1.4.0-premium.2
```

The numeric suffix allows multiple premium builds per stable version (e.g., a
respin with updated agent-intel profiles).

## Feed File

Premium releases are published to `channels/premium.json` with the same schema
as `channels/stable.json` and `channels/beta.json`. Each release entry includes
the `features` field:

```json
{
  "schema_version": 2,
  "channel": "premium",
  "latest_version": "1.3.0-premium.1",
  "releases": [
    {
      "component": "endpoint",
      "version": "1.3.0-premium.1",
      "channel": "premium",
      "released_at": "2026-03-26T10:00:00Z",
      "update_urgency": "recommended",
      "features": ["agent-intel"],
      "artifacts": { ... },
      "detail_url": "https://sixways-ai.github.io/releases/versions/endpoint/1.3.0-premium.1.json"
    }
  ]
}
```

## Artifact Naming

Premium artifacts use a `-premium` infix to distinguish them from standard builds:

| Artifact | Example |
|----------|---------|
| Windows installer | `sixways-endpoint-premium-1.3.0-x64.msi` |
| Windows zip | `sixwaysd-premium-windows-x64.zip` |
| macOS installer | `sixways-endpoint-premium-1.3.0-universal.pkg` |
| macOS binary | `sixwaysd-premium-macos-aarch64.tar.gz` |
| Linux deb | `sixways-endpoint-premium-1.3.0-amd64.deb` |
| Linux rpm | `sixways-endpoint-premium-1.3.0-x86_64.rpm` |
| Linux binary | `sixwaysd-premium-linux-x86_64.tar.gz` |

## Workflow Integration

The `update-feed.yml` workflow handles premium channel releases by:

1. Detecting the `-premium` prerelease tag suffix
2. Assigning the release to the `premium` channel (rather than `beta`)
3. Parsing a `features` key from the YAML frontmatter in the release body
4. Including the `features` array in the feed entry and per-version detail JSON

### Frontmatter for Premium Releases

```markdown
---
urgency: recommended
min_version: 1.2.0
features: agent-intel
sha256_sixwaysd_premium_windows_x64_zip: abc123...
---

## What's New

- Agent Intelligence behavioral monitoring for AI coding agents
```

The `features` frontmatter value is a comma-separated list that gets split into
an array in the feed JSON.

## Licensing

Premium features require a valid subscription license. The endpoint checks the
license on startup and periodically via heartbeat to the licensing service at
`api.sixways.ai`. If the license is missing or expired, premium features are
disabled but the endpoint continues to function with standard capabilities.

## Client Discovery

Endpoints configured for the premium channel poll `channels/premium.json` instead
of `channels/stable.json`. The update logic is identical -- the endpoint downloads
the latest version, verifies the Ed25519 manifest signature, and installs.

Clients that do not have a premium subscription should remain on the `stable`
channel. Channel selection is configured in the endpoint settings file or via
the server's fleet management policy.
