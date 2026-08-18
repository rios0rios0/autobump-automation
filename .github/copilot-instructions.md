# Autobump Automation Repository

This repository provides automated dependency and version management across multiple projects using the [Autobump tool](https://github.com/rios0rios0/autobump). It is a configuration and automation repository containing GitHub Actions workflows that run the Autobump tool daily to manage version bumps and changelog updates across specified projects.

**ALWAYS** reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Test the Repository
- Install required system dependencies:
  - `sudo apt-get update && sudo apt-get install -y curl jq` -- takes 30-60 seconds
- Download and test autobump binary:
  - `curl -fsSL https://raw.githubusercontent.com/rios0rios0/autobump/main/install.sh | sh -s -- --install-dir . --force` -- downloads and installs in <1 second
  - `./autobump --help` -- verify binary works

### Configuration Management
- The main configuration file is `.autobump.yaml` containing:
  - GPG key path for commit signing
  - `exclude_forks`
  - **No** `providers` list. A fine-grained PAT is bound to a single resource owner, so the
    workflow runs one matrix job per owner and appends a single-owner `providers` block to a
    copy of this file at runtime
- Validate configuration syntax: `yamllint .autobump.yaml` -- takes <1 second
  - WARNING: yamllint will report missing document start "---" which is acceptable

### Testing Workflow Components
- Setup minimal git configuration for testing:
  ```bash
  cat <<-EOF >~/.gitconfig
  [user]
      name = Test User
      email = test@example.com
  [push]
      autoSetupRemote = true
  EOF
  ```
- Test autobump discover processing:
  - `./autobump discover` -- fails with authentication errors (<1 second), which is expected without real credentials

## Validation

### Manual Workflow Testing
- **ALWAYS** test workflow components after making configuration changes
- Run the complete workflow simulation:
  1. Download dependencies (30-60 seconds)
  2. Download Autobump binary (<1 second)
  3. Test binary execution (<1 second)
  4. Validate configuration syntax (<1 second)
- **NEVER CANCEL** workflow operations - all steps complete in under 2 minutes
- The actual GitHub Actions workflow runs daily at 18:00 UTC and can be manually triggered

### Configuration Validation Steps
- Validate YAML syntax: `yamllint .autobump.yaml`
- Test autobump config parsing: `./autobump discover` (expect authentication errors)
- Check that secret file paths referenced in config exist in the runner environment

## Common Tasks

### Repository Structure
```
.
├── .autobump.yaml           # Main autobump configuration
├── .editorconfig            # Editor configuration
├── .github/
│   ├── copilot-instructions.md  # This file
│   ├── workflows/
│   │   ├── autobump.yaml        # Daily automation workflow
│   │   ├── claude.yaml           # Claude Code assistant workflow
│   │   ├── claude-code-review.yaml # Claude Code PR review workflow
│   │   └── release.yaml            # Creates Git tag on merge to main
│   ├── pull_request_template/
│   │   ├── bump.md
│   │   └── default.md
│   └── pull_request_template.md
├── CHANGELOG.md             # Release history
├── CLAUDE.md                # Claude Code guidance
├── CONTRIBUTING.md          # Contribution guidelines
├── LICENSE                  # MIT License
└── README.md                # Basic project description
```

### Key Configuration Files

#### .autobump.yaml
Holds only the settings shared by every owner:
```yaml
gpg_key_path: '.secure_files/autobump.asc'
exclude_forks: true
```

The `Render Owner Configuration` step copies it to `${RUNNER_TEMP}/autobump.yaml` and appends
the owner currently being processed, producing the file that `./autobump run --config` reads:
```yaml
providers:
  - type: 'github'
    token: '.secure_files/github_access_token.key'
    organizations:
      - 'medhub-tech'
```

#### .github/workflows/autobump.yaml
GitHub Actions workflow that:
- Runs daily at 18:00 UTC (`cron: '0 18 * * *'`)
- Can be manually triggered via workflow_dispatch
- Fans out into one job per owner via `strategy.matrix.owner`, each entry pairing an owner
  name with the secret holding that owner's fine-grained PAT. `fail-fast: false` keeps one
  owner's broken token from cancelling the others
- Downloads latest Autobump binary via install script
- Configures git with repository variables and secrets
- Validates the GPG key and the owner's token before running
- Renders a single-owner config, then runs `./autobump run --config`
- Asserts the owner was actually reached: AutoBump logs discovery failures and still exits 0,
  so the `Assert Owner Was Reached` step fails the job when the log contains
  `Failed to discover repos in` or no `Discovery complete:` summary
- Cleans up secrets after completion

### Workflow Variables and Secrets Required
The GitHub Actions workflow expects these repository **variables** (`vars.*`):
- `GIT_USER_NAME` - Git commit author name
- `GIT_USER_EMAIL` - Git commit author email
- `GIT_USER_SIGNINGKEY` - GPG signing key ID

The GitHub Actions workflow expects these repository **secrets** (`secrets.*`):
- `GPG_PRIVATE_KEY` - GPG private key for commit signing
- `PERSONAL_ACCESS_TOKEN` - fine-grained PAT for the `rios0rios0` account
- `MEDHUB_TECH_ACCESS_TOKEN` - fine-grained PAT for the `medhub-tech` organization
- `PREFY_ACCESS_TOKEN` - fine-grained PAT for the `prefy` organization

A fine-grained PAT is bound to a single resource owner, so one token cannot cover all three.
Every token's lifetime must be **366 days or less**: both organizations reject longer-lived
fine-grained tokens with `403 ... forbids access via a fine-grained personal access tokens if
the token's lifetime is greater than 366 days`. To add an owner, add a matrix entry and its
secret — no other file changes.

### Expected Timing
- **apt-get update && install**: 30-60 seconds
- **autobump download (install script)**: <1 second
- **configuration validation**: <1 second
- **yamllint validation**: <1 second
- **Complete workflow test**: <2 minutes total

### Common Failure Scenarios
- **"open /home/runner/.gitconfig: no such file or directory"**: Setup git config first
- **"No authentication mechanism implemented"**: Expected when testing without real credentials
- **Missing variables/secrets**: Workflow will fail if repository variables or secrets are not configured

### Making Changes
- **ALWAYS** validate configuration changes with `yamllint .autobump.yaml`
- **ALWAYS** test autobump can parse the config with `./autobump discover` (expect auth errors)
- No build process required - this is a pure configuration repository

### Integration Points
- Main Autobump tool repository: https://github.com/rios0rios0/autobump
- Owners managed by the workflow matrix: `rios0rios0`, `medhub-tech`, `prefy`
- GitHub Actions for automation
- GPG signing for commit verification

### Debugging
- Check workflow runs in GitHub Actions tab
- Review Autobump logs for authentication and processing errors
- Ensure repository variables and secrets are properly configured
- Test configuration changes in a fork before applying to main repository
