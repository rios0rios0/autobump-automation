# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Configuration-only repository that runs [Autobump](https://github.com/rios0rios0/autobump) via GitHub Actions to version-bump repositories owned by `rios0rios0`, `medhub-life` and `prefy`. No build process or source code.

## Key Files

- `.autobump.yaml` — global autobump configuration (GPG key path, `exclude_forks`). It carries **no** `providers` block: the workflow appends a single-owner one per matrix job
- `.github/workflows/autobump.yaml` — daily workflow (cron `0 18 * * *`), one matrix job per owner, runs `./autobump run --config` against the rendered single-owner config
- `.github/workflows/claude.yaml` — Claude Code assistant workflow
- `.github/workflows/claude-code-review.yaml` — Claude Code PR review workflow
- `.github/workflows/release.yaml` — creates a Git tag on merge to `main` (delegates to `rios0rios0/pipelines`)

## Validation

Validate YAML syntax: `yamllint .autobump.yaml`

Test autobump locally:
```bash
curl -fsSL https://raw.githubusercontent.com/rios0rios0/autobump/main/install.sh | sh -s -- --install-dir . --force
./autobump discover  # expects auth errors without real credentials
```

## Workflow Secrets and Variables

Variables (`vars.*`): `GIT_USER_NAME`, `GIT_USER_EMAIL`, `GIT_USER_SIGNINGKEY`

Secrets (`secrets.*`): `GPG_PRIVATE_KEY`, plus one fine-grained PAT per owner —
`PERSONAL_ACCESS_TOKEN` (`rios0rios0`), `MEDHUB_ACCESS_TOKEN` (`medhub-life`), `PREFY_ACCESS_TOKEN` (`prefy`).

A fine-grained PAT is bound to a single resource owner, so one token cannot cover all three.
Each token's lifetime must be **366 days or less** — both organizations reject longer-lived
fine-grained tokens with a 403. Adding an owner means adding a matrix entry in the workflow
and its matching secret; nothing else changes.

## Conventions

- Follow [Development Guide](https://github.com/rios0rios0/guide/wiki) for coding standards and commit conventions.
- Branch naming: `feat/`, `fix/`, `bump/x.x.x` for releases.
- Always validate YAML after config changes.
- Indent with tabs, UTF-8, LF line endings (see `.editorconfig`).
