# Contributing

Contributions are welcome. By participating, you agree to maintain a respectful and constructive environment.

For coding standards, testing patterns, architecture guidelines, commit conventions, and all
development practices, refer to the **[Development Guide](https://github.com/rios0rios0/guide/wiki)**.

## Prerequisites

- [Git](https://git-scm.com/downloads) 2.30+
- A text editor with YAML support
- Basic understanding of [GitHub Actions](https://docs.github.com/en/actions) workflows

## Development Workflow

1. Fork and clone the repository
2. Create a branch: `git checkout -b feat/my-change`
3. Edit the workflow file:
   ```bash
   # The main workflow lives at:
   .github/workflows/autobump.yaml
   ```
4. Edit the Autobump configuration:
   ```bash
   # Configuration file:
   .autobump.yaml
   ```
5. Validate the YAML syntax:
   ```bash
   python3 -c "import yaml; yaml.safe_load(open('.github/workflows/autobump.yaml'))"
   ```
6. Test the workflow manually via GitHub Actions:
   ```bash
   gh workflow run autobump.yaml
   ```
7. Commit following the [commit conventions](https://github.com/rios0rios0/guide/wiki/Life-Cycle/Git-Flow)
8. Open a pull request against `main`
