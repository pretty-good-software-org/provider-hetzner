---
last_validated: 2026-07-04T00:00:00Z
project_type: base-template
---

# Agent Instructions: base-template

This file provides guidance to Claude Code when working with code in this repository.

## Repository Overview

Golden base template for all project templates. Provides the common foundation stack: mise tool management, lefthook git
hooks, mise file-tasks, cog conventional commit enforcement, and shared linter configurations.

Derived templates (template-go, template-nix-config, etc.) are created from this repo and extend it with domain-specific
tooling.

## Repository Structure

```text
base-template/
├── .github/
│   └── workflows/
│       ├── guard.yml
│       ├── lint.yml
│       └── template-guard.yml
├── lefthook/
│   ├── files.yml           # Whitespace, EOF, YAML, large files, merge conflicts
│   ├── lint.yml            # actionlint, yamllint, markdownlint on staged files
│   ├── commit-msg.yml      # Conventional commit enforcement via cog
│   └── secrets.yml         # gitleaks pre-commit and pre-push scans
├── mise-tasks/
│   ├── setup/default       # Install tools and git hooks
│   ├── guard/default       # Template drift guard
│   └── lint/               # default (all), actionlint, yamllint, markdownlint
├── .mise.toml              # Base tools: cocogitto, actionlint, yamllint, markdownlint-cli2
├── .miserc.toml            # Default environment: development
├── .mise.ci.toml           # CI profile (empty override)
├── .mise.development.toml  # Dev profile: lefthook, gitleaks
├── lefthook.yml            # Entry point — extends lefthook/*.yml
├── cog.toml                # Conventional commit config
├── .actionlint.yml         # actionlint config
├── .yamllint.yml           # yamllint config
├── .markdownlint-cli2.jsonc
├── lint-standards.toml     # Authoritative org lint strictness standard
├── CLAUDE.md               # Pointer to this file
└── README.md
```

## Available Commands

```bash
mise run setup:default        # Install dev tools and git hooks
mise run guard                # Validate against the base-template standard
mise run lint:default         # Run all linters (actionlint + yamllint + markdownlint)
mise run lint:actionlint      # Lint GitHub Actions workflows
mise run lint:yamllint        # Lint YAML files
mise run lint:markdownlint    # Lint Markdown files
```

## Conventions

- Conventional commits enforced via cocogitto (`cog verify`) in lefthook commit-msg hook
- Lefthook pre-commit hooks: trailing whitespace, EOF newline, YAML syntax, large files, merge markers, secrets scan,
  linters
- mise layered profiles: base (shared), ci (empty override), development (lefthook + gitleaks)
- mise Style-A pins: every tool version in `.mise.toml`, `.mise.ci.toml`, and `.mise.development.toml` must be exact
  `MAJOR.MINOR.PATCH`
- Org lint ceilings live in `lint-standards.toml`; repos may tighten them, but loosening requires a documented waiver
- mise file-tasks in `mise-tasks/<group>/<name>`, run via `mise run <group>:<name>`; `mise run guard` validates template
  drift
- All tasks use `#!/usr/bin/env bash` + `set -euo pipefail`
- `#MISE description=` header on every task; `#MISE depends=` for aggregator tasks

## Template Guard

The guard is `mise-tasks/guard/default`. It prints `PASS`/`FAIL` for checks `a` through `i` and exits non-zero on any
failure. Check `i` validates lint configs against `lint-standards.toml` for detected languages and rejects `|| true` in
lint tasks. Derived templates should add this reusable workflow caller instead of copying the guard:

```yaml
jobs:
  template-guard:
    uses: pretty-good-software-org/base-template/.github/workflows/template-guard.yml@main
```

Repo-local exceptions can be placed in `.guardrc` (Bash):

- `GUARD_LINT_RUNNER='[self-hosted, Linux, ARM64]'` overrides the expected lint workflow runner.
- `GUARD_SKIP_CHECKS='a,e,h'` skips specific checks by letter, comma-separated. Use `i` only for a documented, temporary
  lint-standard migration waiver.
