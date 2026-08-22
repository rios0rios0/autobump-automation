# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

When a new release is proposed:

1. Create a new branch `bump/x.x.x` (this isn't a long-lived branch!!!);
2. The Unreleased section on `CHANGELOG.md` gets a version number and date;
3. Open a Pull Request with the bump version changes targeting the `main` branch;
4. When the Pull Request is merged, a new Git tag must be created using <LINK TO THE PLATFORM TO OPEN THE PULL REQUEST>.

Releases to productive environments should run from a tagged version.
Exceptions are acceptable depending on the circumstances (critical bug fixes that can be cherry-picked, etc.).

## [Unreleased]

## [0.3.1] - 2026-08-22

### Changed

- changed `.autobump.yaml` to hold only the settings shared by every owner; the `providers` block is now rendered per job from the matrix entry, keeping the owner list and its secret declared in exactly one place
- changed the daily workflow to fan out into one job per owner via `strategy.matrix.owner`, each job authenticating with that owner's own fine-grained PAT (`PERSONAL_ACCESS_TOKEN`, `MEDHUB_ACCESS_TOKEN`, `PREFY_ACCESS_TOKEN`) and running with `fail-fast: false` so one broken token no longer cancels the other owners

### Fixed

- fixed `medhub-tech` and `prefy` never being version-bumped: a single fine-grained PAT is bound to one resource owner, so the shared `PERSONAL_ACCESS_TOKEN` was rejected by both organizations with `403 ... forbids access via a fine-grained personal access tokens if the token's lifetime is greater than 366 days`, and AutoBump logged the discovery failure but still exited `0` — every run since the two owners were added reported success while silently processing only `rios0rios0`
- fixed the workflow reporting success when an owner could not be reached, by adding an `Assert Owner Was Reached` step that fails the job when the run log contains a discovery failure or no `Discovery complete:` summary

## [0.3.0] - 2026-08-13

### Added

- added the `medhub-tech` and `prefy` organizations to the GitHub provider in `.autobump.yaml`, so repositories in both are discovered and version-bumped alongside `rios0rios0`

## [0.2.1] - 2026-05-19

### Changed

- refreshed `CLAUDE.md` and `.github/copilot-instructions.md` to document the new `release.yaml` workflow

## [0.2.0] - 2026-04-28

### Added

- created `CLAUDE.md` with project guidance for Claude Code sessions

### Changed

- refreshed `.github/copilot-instructions.md` to fix cron schedule (13:00 → 18:00 UTC), correct `GIT_USER` variable name to `GIT_USER_NAME`, update workflow command from `discover` to `run`, add `exclude_forks` to config example, and update repository structure tree

## [0.1.1] - 2026-03-30

### Changed

- changed configuration to exclude forked repositories from version bumping

## [0.1.0] - 2026-03-24

The changes weren't tracked until this version.
