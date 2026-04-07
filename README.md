# gh-cli Skill

Comprehensive GitHub CLI (gh) reference skill for AI agents.

## Overview

This skill provides complete documentation and reference for GitHub CLI (`gh`), enabling AI agents to work seamlessly with GitHub from the command line. It covers all major GitHub operations including repositories, issues, pull requests, Actions, projects, releases, gists, codespaces, organizations, and extensions.

## Features

- **Complete Command Reference**: All `gh` commands with examples and options
- **Authentication Guide**: Setup and manage GitHub authentication
- **Repository Management**: Create, clone, fork, and manage repositories
- **Issue Tracking**: Create, list, view, and manage issues
- **Pull Request Workflow**: Full PR lifecycle management
- **GitHub Actions**: Workflow management and CI/CD operations
- **Project Boards**: Kanban-style project management
- **Releases & Packages**: Version management and package publishing
- **Codespaces**: Cloud development environment management
- **Organization Tools**: Team and organization management
- **Extensions**: Extend gh functionality with community extensions

## Installation

### For AI Agents

Install this skill using your agent's skill manager:

```bash
# Using clawhub
clawhub install gh-cli

# Or manually copy the SKILL.md file to your agent's skills directory
```

### GitHub CLI Installation

```bash
# macOS
brew install gh

# Linux
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh

# Windows
winget install --id GitHub.cli

# Verify installation
gh --version
```

## Quick Start

### Authentication

```bash
# Login to GitHub
gh auth login

# Check authentication status
gh auth status
```

### Repository Operations

```bash
# Create a new repository
gh repo create my-project --public --description "My awesome project"

# Clone a repository
gh repo clone owner/repo

# View repository info
gh repo view --web
```

### Issue Management

```bash
# List issues
gh issue list

# Create an issue
gh issue create --title "Bug report" --body-file issue-body.md

# View an issue
gh issue view 123
```

### Pull Requests

```bash
# List pull requests
gh pr list

# Create a pull request
gh pr create --title "Fix bug" --body-file pr-body.md

# Review a pull request
gh pr review 456 --approve
```

### GitHub Actions

```bash
# List workflows
gh workflow list

# Run a workflow
gh workflow run "CI Build"

# View workflow runs
gh run list
```

## Usage Examples

### Daily Development Workflow

```bash
# Check your open PRs
gh pr list --author @me

# Check recent issues assigned to you
gh issue list --assignee @me

# View notifications
gh api notifications

# Create a release
gh release create v1.0.0 --title "Version 1.0.0" --notes "Release notes"
```

### Project Management

```bash
# List projects
gh project list

# Add item to project
gh project item-add 123 --url https://github.com/org/repo/issues/456

# Update item status
gh project item-edit --id ITEM_ID --status-field-id FIELD_ID --single-select-option-id OPTION_ID
```

## Topics

- github-cli
- gh
- ai-agent
- skill
- cli-reference
- automation

## Version

Based on GitHub CLI version 2.85.0 (January 2026)

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

## License

This skill is provided as-is for use with AI agents. Refer to the official GitHub CLI documentation for licensing details.

## Resources

- [Official GitHub CLI Documentation](https://cli.github.com/manual/)
- [GitHub CLI GitHub Repository](https://github.com/cli/cli)
- [GitHub CLI Releases](https://github.com/cli/cli/releases)
