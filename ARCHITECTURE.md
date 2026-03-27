# SixWays Releases — Architecture

Public release feed for all SixWays components (endpoint, server, gateway, ml-gateway, sandbox, vscode-extension).

## High-Level Design

Releases provides a static JSON feed of endpoint software releases, hosted on GitHub Pages. A GitHub Actions workflow regenerates the feed whenever a new release is published. The feed supports multiple channels (stable, beta) and per-version detail pages with artifact metadata.

## Project Structure

```
releases/
  index.json              # Complete release catalog (all components)
  latest.json             # Latest release per component
  channels/
    stable.json           # Stable channel releases
    beta.json             # Beta channel releases
  components/
    {component}/
      latest.json         # Per-component latest stable/beta
  versions/
    {component}/
      {version}.json      # Per-version detail with artifacts
```

## Key Abstractions

- **JSON feed schema (version 2)**: Each feed file includes a `schema_version` field for forward compatibility as the schema evolves.
- **Channel model**: Stable and beta channels with separate feed files. Clients subscribe to a channel and poll its feed.
- **Asset naming convention**: Platform, architecture, and package type are parsed from artifact filenames — `*.pkg` for macOS, `*.msi` for Windows, `*.deb` / `*.rpm` for Linux.
- **YAML frontmatter in GitHub Release body**: Release notes include structured metadata (urgency, min_version, containers) parsed from YAML frontmatter in the GitHub Release description.
- **GitHub Actions workflow**: Automatically regenerates all JSON feed files when a GitHub Release is published.

## Cross-Repo Dependencies

- Consumed by **sixways-server** for upstream release discovery and fleet update management.
- Consumed by **sixways-endpoint** for self-update checks.
- Consumed by **sixways-gateway** and **ml-gateway** for version awareness.
- Consumed by **vscode-extension** for extension update checks.

## Design Decisions

- **Static JSON on GitHub Pages over an API endpoint** — Zero infrastructure to operate. The feed is a set of static files served by GitHub's CDN.
- **Semantic versioning** — All releases follow semver for predictable upgrade compatibility.
- **Channel-based release model** — Separates stable and beta audiences, allowing staged rollouts and early access testing.

## See Also

- [System Overview](https://github.com/sixways-ai/architecture/blob/main/docs/system-overview.md) (internal -- may require access)
- [Repository Map](https://github.com/sixways-ai/architecture/blob/main/docs/repo-map.md) (internal -- may require access)
