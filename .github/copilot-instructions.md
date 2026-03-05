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
  - A `providers` list with provider type, token path, and target organizations
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
- The actual GitHub Actions workflow runs daily at 13:00 UTC and can be manually triggered

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
│   │   └── autobump.yaml    # Daily automation workflow
│   ├── pull_request_template/
│   └── pull_request_template.md
├── CONTRIBUTING.md          # Contribution guidelines
├── LICENSE                  # MIT License
└── README.md                # Basic project description
```

### Key Configuration Files

#### .autobump.yaml
Contains authentication tokens and provider configuration:
```yaml
gpg_key_path: '.secure_files/autobump.asc'

providers:
  - type: 'github'
    token: '.secure_files/github_access_token.key'
    organizations:
      - 'rios0rios0'
```

#### .github/workflows/autobump.yaml
GitHub Actions workflow that:
- Runs daily at 13:00 UTC (`cron: '0 13 * * *'`)
- Can be manually triggered via workflow_dispatch
- Downloads latest Autobump binary via install script
- Configures git with repository variables and secrets
- Runs `./autobump discover`

### Workflow Variables and Secrets Required
The GitHub Actions workflow expects these repository **variables** (`vars.*`):
- `GIT_USER` - Git commit author name
- `GIT_USER_EMAIL` - Git commit author email
- `GIT_USER_SIGNINGKEY` - GPG signing key ID

The GitHub Actions workflow expects these repository **secrets** (`secrets.*`):
- `GPG_PRIVATE_KEY` - GPG private key for commit signing
- `PERSONAL_ACCESS_TOKEN` - GitHub personal access token for API authentication

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
- Target GitHub organization managed by providers config: `rios0rios0`
- GitHub Actions for automation
- GPG signing for commit verification

### Debugging
- Check workflow runs in GitHub Actions tab
- Review Autobump logs for authentication and processing errors
- Ensure repository variables and secrets are properly configured
- Test configuration changes in a fork before applying to main repository
