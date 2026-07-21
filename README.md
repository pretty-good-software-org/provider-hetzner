# base-template

Golden base template for all project templates. Provides the common foundation: mise tool management, lefthook git
hooks, mise tasks, and linter configs.

## What's Included

| Layer          | Files                                                                                 | Purpose                                                 |
| -------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| **mise**       | `.mise.toml`, `.miserc.toml`, `.mise.ci.toml`, `.mise.development.toml`               | Tool version management with layered profiles           |
| **lefthook**   | `lefthook.yml`, `lefthook/*.yml`                                                      | Git hooks for formatting, linting, secrets, commits     |
| **mise tasks** | `mise-tasks/<group>/<task>`                                                           | Namespaced file-tasks run via `mise run <group>:<task>` |
| **linters**    | `.actionlint.yml`, `.yamllint.yml`, `.markdownlint-cli2.jsonc`, `lint-standards.toml` | Linter configurations and org strictness ceilings       |
| **cocogitto**  | `cog.toml`                                                                            | Conventional commit message enforcement                 |

## Prerequisites

- [mise](https://mise.jdx.dev/) — tool version manager

## Quick Start

```bash
# Install tools and git hooks
mise run setup:default

# Run all linters
mise run lint:default

# Validate this repo against the base-template standard
mise run guard
```

`setup:default` installs the pinned Python and uv tools, then runs `setup:mdformat`. That task prepares the dedicated
mdformat environment from `mdformat-requirements.lock` with `uv pip sync --require-hashes`. The environment lives at
`${XDG_CACHE_HOME:-$HOME/.cache}/mise-mdformat`, outside the repository, and Mise adds its `bin` directory to `PATH`.
The path includes hashes of the repository root, requirements lock, and the complete `.mise.toml` plus `mise.lock` tool
provenance. A single synchronous Python supervisor owns the cache lease and installer process group. It builds one
completion-marked immutable generation from random staging, atomically publishes the deterministic completed generation
through an `active` pointer, and removes orphan staging before reuse. A lock owner is considered dead only after three
consecutive process lookup misses, preventing a transient `ps` failure from exposing a live staging generation to
cleanup. The completion marker, manifest, package set, current-UID ownership, 0700/0600 permissions, and non-traversing
path checks are required before reuse. The current requirements lock is the interim format. PEP 751 `pylock.toml` is the
documented migration path when the canonical uv project export is adopted; it does not replace the pinned Mise Python or
uv bootstrap.

The cache threat boundary is intentionally local and same-UID: a malicious process with this UID is trusted because it
can replace any local executable or secret, including the Mise-managed tools. These checks detect accidental, stale, or
incomplete cache state; they do not provide same-UID tamper resistance.

The mdformat lifecycle is POSIX-only and supports Linux ARM64, Linux x64 including glibc and musl, and macOS ARM64 and
x64. Windows is not supported: the canonical lock files intentionally contain no Windows artifacts, and the reproducible
lock command targets only the supported platforms:

```bash
mise lock --platform linux-arm64,linux-arm64-musl,linux-x64,linux-x64-musl,macos-arm64,macos-x64
```

Each requirements-lock and Mise-tool identity has its own content-addressed cache parent. Complete generations and their
parents are retained because a reader may already have resolved the old `active` target when a writer publishes a new
pointer. Storage therefore grows when lock or tool identities change; there is no unsafe age-based garbage collection.
An operator may prune the whole `mise-mdformat` cache only while no mdformat setup, format, check, lint, or hook task is
running. The next setup recreates the cache. Never delete a parent by age while a mdformat task may still be using it.

Run the focused supply-chain tests with:

```bash
mise run test:mdformat
```

## Creating a Derived Template

1. Create a new repo from this template via GitHub "Use this template"
2. Add domain-specific tools to `.mise.toml` using Style-A exact pins (for example, `go = "1.24.4"`; never `latest`,
   `1`, or `1.24`)
3. Add domain-specific lefthook hooks in `lefthook/` and extend `lefthook.yml` with explicit file names, not globs
4. Add domain-specific mise tasks in `mise-tasks/<group>/<task>`
5. Add domain-specific `.gitignore` entries
6. Add the template guard caller workflow shown below
7. Update `CLAUDE.md` and `README.md` for your domain

## Template Drift Guard

Run the guard locally before opening changes in any derived template:

```bash
mise run guard
```

Derived templates should call the reusable guard from `base-template` instead of copying the guard script:

```yaml
name: Guard

on:
  pull_request:
  push:
    branches: [main]

jobs:
  template-guard:
    uses: pretty-good-software-org/base-template/.github/workflows/template-guard.yml@main
```

The reusable workflow checks out the calling repository, fetches `mise-tasks/guard/default` from `base-template@main`,
and runs it as the single source of truth.

Check `i` reads `lint-standards.toml` from the current repo when present, or from the shipped guard action when used by
derived repositories. It verifies detected Go, Rust, Python, and YAML lint thresholds and rejects lint tasks that
swallow failures with `|| true`.

### `.guardrc` overrides

If a derived template needs a repo-local exception, create `.guardrc` as a Bash file. Supported variables:

- `GUARD_LINT_RUNNER='[self-hosted, Linux, ARM64]'` overrides the expected `.github/workflows/lint.yml` runner.
- `GUARD_SKIP_CHECKS='a,e,h'` skips specific guard checks by letter, comma-separated.

Keep overrides temporary and documented in the derived template.

## Structure

```text
.
├── .mise.toml                 # Base tools (cocogitto, actionlint, yamllint, markdownlint)
├── .miserc.toml               # Default environment (development)
├── .mise.ci.toml              # CI profile (empty — uses base only)
├── .mise.development.toml     # Dev profile (lefthook, gitleaks)
├── lefthook.yml               # Git hooks entry point
├── lefthook/
│   ├── files.yml              # Whitespace, EOF, YAML, large files, merge conflicts
│   ├── lint.yml               # actionlint, yamllint, markdownlint
│   ├── commit-msg.yml         # Conventional commit enforcement
│   └── secrets.yml            # gitleaks pre-commit and pre-push
├── mise-tasks/
│   ├── setup/default          # Install tools and hooks
│   ├── setup/mdformat         # Rebuild hash-locked mdformat environment
│   ├── setup/mdformat-env     # Derive and activate the environment path
│   ├── setup/mdformat-worker  # Run hash-locked installation in a process group
│   ├── setup/mdformat-supervisor # Own the lock and installer lifecycle
│   ├── guard/default          # Template drift guard
│   ├── lint/                  # default (all), actionlint, yamllint, markdownlint
│   ├── test/mdformat          # mdformat supply-chain regression tests
│   └── test/mdformat-readers  # concurrent reader publication tests
├── lint-standards.toml        # Authoritative org lint strictness standard
├── .actionlint.yml            # actionlint config
├── .yamllint.yml              # yamllint config
├── .markdownlint-cli2.jsonc   # markdownlint config
├── mdformat-requirements.in   # Direct mdformat package pins
├── mdformat-requirements.lock # Hash-locked mdformat packages
├── cog.toml                   # cocogitto conventional commit config
├── .gitignore                 # Common ignores (IDE, OS, Node, Secrets)
├── CLAUDE.md                  # Agent instructions
└── README.md                  # This file
```

## Conventions

- Conventional commits (`feat:`, `fix:`, `chore:`)
- Lefthook pre-commit hooks enforce formatting and secrets scanning
- mise layered profiles: base (CI + dev), ci (empty override), development (extra dev tools)
- mise Style-A pins: every tool version in `.mise*.toml` is exact `MAJOR.MINOR.PATCH`
- Lint ceilings live in `lint-standards.toml`; derived repos may tighten them but need a documented waiver to loosen
  them
- mise file-tasks with namespaced groups (`setup:`, `lint:`, `guard`)
