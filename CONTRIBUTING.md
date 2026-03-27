# Contributing to SixWays Releases

SixWays Releases manages release metadata, version schemas, and distribution workflows for the SixWays platform. We welcome contributions that improve schema definitions and automation.

## Table of Contents

- [How to Contribute](#how-to-contribute)
- [Development Setup](#development-setup)
- [Code Style](#code-style)
- [Testing](#testing)
- [Pull Request Process](#pull-request-process)
- [Commit Conventions](#commit-conventions)
- [License](#license)

---

## How to Contribute

We accept contributions in the following areas:

- **Schema Improvements**: Refine JSON schemas for release metadata, version manifests, and update definitions
- **Workflow Enhancements**: Improve GitHub Actions workflows for build, publish, and distribution
- **Documentation**: Improve release process guides and schema documentation

---

## Development Setup

No special tooling is required beyond a text editor and Git.

```bash
git clone https://github.com/sixways-ai/releases.git
cd releases
```

For full development environment setup, see [docs/development.md](docs/development.md).

---

## Code Style

- **JSON**: Valid JSON with 2-space indentation
- **YAML**: Valid YAML with 2-space indentation, consistent quoting
- Keep schema files well-organized with clear property descriptions
- Use descriptive key names that are self-documenting
- Maintain alphabetical ordering of properties where practical

---

## Testing

Validate JSON files:

```bash
# Validate all JSON files are syntactically correct
find . -name "*.json" -exec python -m json.tool {} > /dev/null \;
```

Verify GitHub Actions workflows:

```bash
# Lint workflow files
npx yaml-lint .github/workflows/*.yml

# Optionally use act to test workflows locally
# https://github.com/nektos/act
act -n
```

Ensure all validation passes before submitting a pull request.

---

## Pull Request Process

1. **Single Responsibility**: Each PR should address one logical change
2. **CI Passing**: All GitHub Actions checks must pass
3. **Description**: Provide a clear explanation of the change and its rationale
4. **Approval**: Requires 1 approval from a maintainer
5. **Merge**: Squash and merge to maintain clean history

### Review Criteria

- Schema changes are backwards-compatible or versioned appropriately
- Workflow changes are tested and do not break existing release processes
- JSON and YAML files are valid and well-structured

---

## Commit Conventions

Use [Conventional Commits](https://www.conventionalcommits.org/) style:

```
feat: add platform field to release manifest schema
fix: correct version constraint format in update schema
docs: document release channel definitions
chore: update actions/checkout to v4
refactor: consolidate platform-specific workflow steps
```

Keep the subject line under 72 characters. Use the body for additional context when needed.

---

## License

By contributing to SixWays Releases, you agree that your contributions will be licensed under the Apache License 2.0.

For details, see [LICENSE](LICENSE).

---

## Questions and Support

- **Bug Reports**: Open a GitHub Issue
- **General Questions**: Open a GitHub Discussion
- **Security Concerns**: Email security@sixways.ai

Thank you for contributing to SixWays Releases.
