---
name: gh-cli
description: GitHub CLI (gh) comprehensive reference for repositories, issues, pull requests, Actions, projects, releases, gists, codespaces, organizations, extensions, and all GitHub operations from the command line.
---

# GitHub CLI (gh) - Quick Reference

Comprehensive reference for GitHub CLI (gh) - work seamlessly with GitHub from the command line.

**Version:** 2.85.0 (current as of January 2026)

## Quick Start

### Installation

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

### Authentication

```bash
# Interactive login
gh auth login

# Check status
gh auth status

# Setup git integration
gh auth setup-git
```

## CLI Structure Overview

The GitHub CLI provides commands for all major GitHub features:

- **auth** - Authentication and account management
- **repo** - Repository operations (create, clone, fork, etc.)
- **issue** - Issue management
- **pr** - Pull request workflows
- **run/workflow** - GitHub Actions
- **project** - GitHub Projects
- **release** - Release management
- **gist** - Gist operations
- **codespace** - Codespaces management
- **search** - Search across GitHub
- **api** - Direct API access
- And many more (extensions, labels, ssh-key, etc.)

## Detailed References

This skill is organized into modular reference documents. Load only what you need:

### Core Operations

- **[Authentication](references/auth.md)** - Login, token management, account switching
- **[Repositories](references/repo.md)** - Create, clone, fork, edit, delete repos
- **[Issues](references/issues.md)** - Create, list, comment, manage issues
- **[Pull Requests](references/prs.md)** - PR workflows, reviews, merges ⚠️ **Critical warnings inside**
- **[PR Reviews (Extension)](references/pr-reviews.md)** - Inline code review, review threads, gh-pr-review extension

### Automation & CI/CD

- **[GitHub Actions](references/actions.md)** - Workflows, runs, secrets, variables
- **[Search](references/search.md)** - Code, commits, issues, PRs, repos search

### Project Management

- **[Projects](references/projects.md)** - GitHub Projects management
- **[Releases](references/releases.md)** - Release creation and asset management

### Development Tools

- **[Gists](references/gists.md)** - Gist creation and management
- **[Codespaces](references/codespaces.md)** - Remote development environments

### Advanced Features

- **[API & Advanced](references/advanced.md)** - API calls, extensions, aliases, labels, SSH/GPG keys
- **[Configuration](references/config.md)** - Environment setup, output formatting, best practices

## Critical Best Practices

### 1. Always Use `--body-file` for PR/Issue Content

**NEVER use `--body` parameter!** Command-line argument parsing fails with special characters, especially backticks (`) used in code blocks. This causes escaping issues that result in garbled text or parsing errors.

```bash
# ✅ CORRECT, workspace_dir is the workspace root (top-level directory containing all repositories). Exception: if workspace itself is a repository root, use that directory.
gh pr create --title "Feature" --body-file workspace_dir/temp/description.md

# ❌ WRONG - backticks and special chars cause escaping issues
gh pr create --title "Feature" --body "Use `code()` function..."
```

### 2. Export PR Content to Files Before Viewing

**NEVER view PR comments directly in terminal!** Output may be truncated.

```bash
# ✅ CORRECT workflow, workspace_dir is the workspace root (top-level directory containing all repositories). Exception: if workspace itself is a repository root, use that directory.
gh pr view 123 --comments > workspace_dir/temp/pr-123.md
cat workspace_dir/temp/pr-123.md

# ❌ WRONG (may lose data)
gh pr view 123 --comments
```

### 3. PowerShell Redirection Warning

In PowerShell, only `>` redirection is reliable:

```powershell
# ✅ CORRECT
gh pr view 123 --comments > workspace_dir/temp/pr-123.md

# ❌ WRONG
gh pr view 123 --comments | Out-File workspace_dir/temp/pr-123.md
gh pr view 123 --comments > workspace_dir/temp/pr-123.md 2>&1
```

## Common Workflows

### Create PR from Issue

```bash
gh issue develop 123 --branch feature/issue-123
# Make changes, commit, push
git add . && git commit -m "Fix #123" && git push
gh pr create --title "Fix #123" --body-file workspace_dir/temp/pr-description.md
```

### Fork Sync Workflow

```bash
gh repo fork original/repo --clone
cd repo
git remote add upstream https://github.com/original/repo.git
gh repo sync
```

### CI/CD Monitoring

```bash
RUN_ID=$(gh workflow run ci.yml --ref main --jq '.databaseId')
gh run watch "$RUN_ID"
gh run download "$RUN_ID" --dir ./artifacts
```

## Getting Help

```bash
# General help
gh --help

# Command-specific help
gh pr --help
gh issue create --help

# Help topics
gh help formatting
gh help environment
```

## When to Use This Skill

Use this skill when you need to:
- Automate GitHub operations from the command line
- Manage repositories, issues, and pull requests efficiently
- Interact with GitHub Actions workflows and runs
- Work with GitHub Projects, Releases, or Codespaces
- Make direct API calls to GitHub
- Search across GitHub code, issues, or repositories
- Manage GitHub authentication and configurations

For detailed command references, see the linked documentation files above.
