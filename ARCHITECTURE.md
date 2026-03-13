# SixWays Releases — Architecture

Public release feed for endpoint software updates.

## High-Level Design

Releases provides a static JSON feed of endpoint software releases, hosted on GitHub Pages. A GitHub Actions workflow regenerates the feed whenever a new release is published. The feed supports multiple channels (stable, beta) and per-version detail pages with artifact metadata.

## Project Structure

```
releases/
  index.json              # Complete release catalog
  latest.json             # Latest stable release with full detail
  channels/
    stable.json           # Stable channel releases
    beta.json             # Beta channel releases
  versions/
    {version}.json        # Per-version detail with artifact list
```

## Key Abstractions

- **JSON feed schema (version 1)**: Each feed file includes a `schema_version` field for forward compatibility as the schema evolves.
- **Channel model**: Stable and beta channels with separate feed files. Clients subscribe to a channel and poll its feed.
- **Asset naming convention**: Platform, architecture, and package type are parsed from artifact filenames — `*.pkg` for macOS, `*.msi` for Windows, `*.deb` / `*.rpm` for Linux.
- **YAML frontmatter in GitHub Release body**: Release notes include structured metadata (urgency, min_version, containers) parsed from YAML frontmatter in the GitHub Release description.
- **GitHub Actions workflow**: Automatically regenerates all JSON feed files when a GitHub Release is published.

## Cross-Repo Dependencies

- Consumed by **sixways-server** for upstream release discovery and fleet update management.

## Design Decisions

- **Static JSON on GitHub Pages over an API endpoint** — Zero infrastructure to operate. The feed is a set of static files served by GitHub's CDN.
- **Semantic versioning** — All releases follow semver for predictable upgrade compatibility.
- **Channel-based release model** — Separates stable and beta audiences, allowing staged rollouts and early access testing.

## See Also

- [System Overview](https://github.com/sixways-ai/architecture/blob/main/docs/system-overview.md)
- [Repository Map](https://github.com/sixways-ai/architecture/blob/main/docs/repo-map.md)
