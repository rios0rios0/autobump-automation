# Autobump Automation Repository

This repository provides automated dependency and version management across multiple projects using the [autobump tool](https://github.com/rios0rios0/autobump). It is a configuration and automation repository containing GitHub Actions workflows that run the autobump tool daily to manage version bumps and changelog updates across specified projects.

**ALWAYS** reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Test the Repository
- Install required system dependencies:
  - `sudo apt-get update && sudo apt-get install -y unzip curl jq` -- takes 30-60 seconds
- Download and test autobump binary:
  - `export AUTOBUMP_VERSION=$(curl -sSLH 'Accept: application/vnd.github.v3+json' 'https://api.github.com/repos/rios0rios0/autobump/releases/latest' | jq -r '.tag_name')`
  - `curl -LO "https://github.com/rios0rios0/autobump/releases/download/$AUTOBUMP_VERSION/autobump-$AUTOBUMP_VERSION.zip"` -- downloads ~4.4MB in <1 second
  - `unzip "autobump-$AUTOBUMP_VERSION.zip"` -- extracts in <1 second
  - `./autobump --help` -- verify binary works

### Configuration Management
- The main configuration file is `.autobump.yaml` containing:
  - Authentication token paths (GPG key, GitHub access token)
  - List of projects to manage (6 repositories by default)
- **CRITICAL**: Configuration format uses `gitlab_access_token` or `azure_devops_access_token` keys, NOT `github_access_token`
- **CRITICAL**: Current .autobump.yaml has incorrect format and will fail with "field github_access_token not found" error
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
- Test autobump batch processing:
  - `./autobump batch -c .autobump.yaml` -- **WILL FAIL** with config format error
  - Expected error: "field github_access_token not found in type main.GlobalConfig"
  - With corrected config format: fails with authentication errors (<1 second), which is expected

## Validation

### Manual Workflow Testing
- **ALWAYS** test workflow components after making configuration changes
- Run the complete workflow simulation:
  1. Download dependencies (30-60 seconds)
  2. Download autobump binary (<1 second)
  3. Test binary execution (<1 second)
  4. Validate configuration syntax (<1 second)
- **NEVER CANCEL** workflow operations - all steps complete in under 2 minutes
- The actual GitHub Actions workflow runs daily at midnight UTC and can be manually triggered

### Configuration Validation Steps
- Validate YAML syntax: `yamllint .autobump.yaml`
- Test autobump config parsing: `./autobump batch -c .autobump.yaml` (expect authentication errors)
- Verify all project URLs are accessible and valid GitHub repositories
- Check that secret file paths referenced in config exist in the runner environment

## Common Tasks

### Repository Structure
```
.
├── .autobump.yaml           # Main autobump configuration
├── .editorconfig           # Editor configuration
├── .github/
│   ├── workflows/
│   │   └── bump.yaml       # Daily automation workflow
│   └── pull_request_template.md
├── CHANGELOG.md            # Currently empty
├── LICENSE                 # MIT License
└── README.md              # Basic project description
```

### Key Configuration Files

#### .autobump.yaml
Contains authentication tokens and project list:
```yaml
# CURRENT FILE HAS INCORRECT FORMAT - WILL FAIL
gpg_key_path: ".secure_files/autobump.asc"
github_access_token: ".secure_files/github_access_token.key"  # WRONG KEY NAME

# CORRECT FORMAT SHOULD BE:
gitlab_access_token: ".secure_files/github_access_token.key"  # Correct key name
gpg_key_path: ".secure_files/autobump.asc"

projects:
  - path: "https://github.com/rios0rios0/autobump.git"
  - path: "https://github.com/rios0rios0/terra.git"
  - path: "https://github.com/rios0rios0/pipelines.git"
  - path: "https://github.com/rios0rios0/versainit.git"
  - path: "https://github.com/rios0rios0/rios0rios0.git"
  - path: "https://github.com/rios0rios0/boss.git"
```

#### .github/workflows/bump.yaml
GitHub Actions workflow that:
- Runs daily at midnight UTC (`cron: '0 0 * * *'`)
- Can be manually triggered via workflow_dispatch
- Downloads latest autobump binary
- Configures git with secrets
- Runs `./autobump batch`

### Workflow Secrets Required
The GitHub Actions workflow expects these repository secrets:
- `GIT_USER` - Git commit author name
- `GIT_USER_EMAIL` - Git commit author email
- `GIT_USER_SIGNINGKEY` - GPG signing key ID (optional)

### Expected Timing
- **apt-get update && install**: 30-60 seconds
- **autobump download**: <1 second (4.4MB download)
- **autobump extraction**: <1 second
- **configuration validation**: <1 second
- **yamllint validation**: <1 second
- **Complete workflow test**: <2 minutes total

### Common Failure Scenarios
- **"field github_access_token not found"**: Current .autobump.yaml has this error - use `gitlab_access_token` or `azure_devops_access_token` instead
- **"open /home/runner/.gitconfig: no such file or directory"**: Setup git config first
- **"No authentication mechanism implemented"**: Expected when testing without real credentials
- **Missing secrets**: Workflow will fail if repository secrets are not configured

### Making Changes
- **CRITICAL**: Current .autobump.yaml has incorrect config format and needs to be fixed
- To fix the configuration error:
  1. Change `github_access_token:` to `gitlab_access_token:` in .autobump.yaml
  2. This is the only change needed - the file path value stays the same
- **ALWAYS** validate configuration changes with `yamllint .autobump.yaml`
- **ALWAYS** test autobump can parse the config with `./autobump batch -c .autobump.yaml`
- **ALWAYS** verify project URLs are valid and accessible
- Update CHANGELOG.md when making configuration changes (per PR template)
- No build process required - this is a pure configuration repository

### Integration Points
- Main autobump tool repository: https://github.com/rios0rios0/autobump
- Managed project repositories listed in .autobump.yaml
- GitHub Actions for automation
- GPG signing for commit verification (optional)

### Debugging
- Check workflow runs in GitHub Actions tab
- Review autobump logs for authentication and processing errors
- Validate that all referenced projects exist and are accessible
- Ensure repository secrets are properly configured
- Test configuration changes in a fork before applying to main repository
