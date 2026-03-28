# SixWays Releases

Public release feed for all SixWays components (endpoint, server, gateway, ml-gateway, sandbox, vscode-extension).

## Stack

- **Format**: Static JSON files served via GitHub Pages
- **CI**: GitHub Actions (auto-generates feeds on release publish)
- **Base URL**: `https://sixways-ai.github.io/releases`

## Structure

```
index.json                          # Complete release catalog (all components)
latest.json                         # Latest release per component
channels/
  stable.json                       # Stable channel
  beta.json                         # Beta channel
  premium.json                      # Premium channel (agent-intel feature builds)
components/
  {component}/
    latest.json                     # Per-component latest stable/beta
versions/
  {component}/
    {version}.json                  # Per-version detail with artifacts
```

## How It Works

1. Create a GitHub Release with component-prefixed tag (e.g., `server/v1.3.0`, `endpoint/v1.0.0`)
2. For endpoint: attach binary artifacts (pkg, deb, rpm, msi, tar.gz)
3. For Docker components (server, gateway, ml-gateway, sandbox): add `docker_image` and `docker_digest` in YAML frontmatter
4. For vscode-extension: add `vsix_url` in YAML frontmatter
5. Legacy bare tags (e.g., `v1.0.0`) are treated as endpoint releases
6. GitHub Actions workflow regenerates all JSON feed files

## Schema

- Current schema version: 2
- All JSON files include `schema_version` field
- Tags use `{component}/v{version}` format
- Components: endpoint, server, gateway, ml-gateway, sandbox, vscode-extension
- Channels: stable, beta, premium (premium uses `-premium.N` prerelease suffix)

## Consumed By

SixWays Server polls this feed to discover upstream releases and manage fleet updates. The gateway and other components also check for their own updates.
