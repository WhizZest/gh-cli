# GitHub CLI & Git Workflow Skills

AI agent skills for GitHub operations and Git workflow best practices.

## Overview

This repository contains two complementary skills for AI agents working with Git and GitHub:

1. **[gh-cli](#gh-cli-skill)** - Comprehensive GitHub CLI (`gh`) reference covering all major GitHub operations
2. **[git-commit-push](#git-commit-push-skill)** - Git commit & push workflow guidelines ensuring safe, atomic commits with user approval

## Skills

### gh-cli Skill

Comprehensive GitHub CLI (gh) reference skill for AI agents.

**Description:** Complete documentation and reference for GitHub CLI (`gh`), enabling AI agents to work seamlessly with GitHub from the command line. Covers repositories, issues, pull requests, Actions, projects, releases, gists, codespaces, organizations, and extensions.

**Key Features:**
- **Modular Design**: SKILL.md serves as a navigation hub (189 lines), with detailed content in 12 modular reference documents
- **Progressive Disclosure**: Load only the documentation you need, reducing context window usage by 92%
- **Complete Command Reference**: All `gh` commands with examples and options
- **Best Practices**: Critical warnings and workflows for common pitfalls

**Structure:**
```
skills/gh-cli/
├── SKILL.md                    # Main navigation hub
└── references/                 # Modular reference documents
    ├── auth.md                 # Authentication & account management
    ├── repo.md                 # Repository operations
    ├── issues.md               # Issue management
    ├── prs.md                  # Pull request workflows ⚠️ Critical warnings
    ├── actions.md              # GitHub Actions (runs, workflows, secrets)
    ├── projects.md             # GitHub Projects
    ├── releases.md             # Release management
    ├── gists.md                # Gist operations
    ├── codespaces.md           # Codespaces management
    ├── search.md               # Search functionality
    ├── advanced.md             # API, extensions, aliases, labels, keys
    └── config.md               # Configuration & environment setup
```

**Features:**
- Authentication Guide ([auth.md](skills/gh-cli/references/auth.md))
- Repository Management ([repo.md](skills/gh-cli/references/repo.md))
- Issue Tracking ([issues.md](skills/gh-cli/references/issues.md))
- Pull Request Workflow ([prs.md](skills/gh-cli/references/prs.md)) ⚠️ **Critical warnings inside**
- GitHub Actions ([actions.md](skills/gh-cli/references/actions.md))
- Project Boards ([projects.md](skills/gh-cli/references/projects.md))
- Releases & Packages ([releases.md](skills/gh-cli/references/releases.md))
- Codespaces ([codespaces.md](skills/gh-cli/references/codespaces.md))
- Search ([search.md](skills/gh-cli/references/search.md))
- Organization Tools ([advanced.md](skills/gh-cli/references/advanced.md))
- Extensions & API Access ([advanced.md](skills/gh-cli/references/advanced.md))

### git-commit-push Skill

Git commit & push workflow guidelines ensuring safe, user-approved commits.

**Description:** 规范化Git提交与推送流程，确保遵循用户审核、原子提交、精准文件选择等核心原则。基于实际错误案例总结的最佳实践。

**Core Principles:**

1. **User Approval Required** - Never execute `git commit` or `git push` without explicit user consent. Each operation requires separate approval.
2. **Atomic Commits** - One logical change per commit. Complete a feature, then ask to commit.
3. **Precise File Selection** - Never use `git add -A` or `git add .`. Always specify exact files to commit.
4. **Temporary File Management** - Use `<workspace_dir>/temp/` directory for temporary files; never commit them accidentally.
5. **Structured Commit Messages** - Follow conventional commit format; use temp files for multi-line messages.

**Key Guidelines:**
```bash
# ✅ CORRECT: Specify files explicitly
git add src/file1.ts src/file2.ts

# ❌ WRONG: Adds all files including temporaries
git add -A && git commit -m "..."

# ✅ CORRECT: Check before committing
git status
git diff --cached

# ✅ CORRECT: Multi-line commit messages via file
git commit --file=<workspace_dir>/temp/git-commit-msg.txt
```

## Repository Structure

```
skills/
├── gh-cli/                     # GitHub CLI reference skill
│   ├── SKILL.md                # Navigation hub
│   └── references/             # 12 modular reference documents
│       ├── auth.md
│       ├── repo.md
│       ├── issues.md
│       ├── prs.md
│       ├── actions.md
│       ├── projects.md
│       ├── releases.md
│       ├── gists.md
│       ├── codespaces.md
│       ├── search.md
│       ├── advanced.md
│       └── config.md
└── git-commit-push/            # Git workflow guidelines skill
    └── SKILL.md                # Commit/push best practices
```

## Installation

### For AI Agents

Install skills using your agent's skill manager:

```bash
# Using skills.sh (install both skills)
npx skills add https://github.com/WhizZest/agent-git-skills

# Or manually copy the skill directories to your agent's skills directory
```

### Prerequisites

#### GitHub CLI (for gh-cli skill)

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

#### Git (for git-commit-push skill)

Git must be installed and configured with user identity:

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

## Quick Start

### Using gh-cli Skill

#### Authentication

```bash
gh auth login
gh auth status
```

#### Repository Operations

```bash
gh repo create my-project --public --description "My awesome project"
gh repo clone owner/repo
gh repo view --web
```

#### Issue & PR Management

```bash
gh issue list
gh issue create --title "Bug report" --body-file issue-body.md
gh pr list
gh pr create --title "Fix bug" --body-file pr-body.md
```

### Using git-commit-push Skill

The `git-commit-push` skill should be invoked automatically when an AI agent needs to commit or push changes. It enforces:

1. **Ask user before committing** - Show which files will be committed
2. **Get approval on commit message** - Let user review the message content
3. **Specify files explicitly** - Never use `git add -A`
4. **Use temp directory** - Keep temporary files out of commits

## Critical Best Practices

### ⚠️ Always Use `--body-file` for PR/Issue Content

**NEVER use `--body` parameter!** Command-line argument parsing fails with special characters, especially backticks (\`) used in code blocks.

```bash
# ✅ CORRECT
gh pr create --title "Feature" --body-file description.md

# ❌ WRONG - backticks cause escaping issues
gh pr create --title "Feature" --body "Use \`code()\` function..."
```

### ⚠️ Export PR Content to Files Before Viewing

**NEVER view PR comments directly in terminal!** Terminal output may be truncated, causing loss of important information.

```bash
# ✅ CORRECT workflow
gh pr view 123 --comments > pr-123.md
cat pr-123.md

# ❌ WRONG (may lose data)
gh pr view 123 --comments
```

See [prs.md](skills/gh-cli/references/prs.md) for more details.

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

## How the Skills Work Together

These two skills are designed to complement each other:

| Scenario | Skill Used | Purpose |
|----------|-----------|---------|
| Clone/create repos | **gh-cli** | Repository operations |
| Manage issues & PRs | **gh-cli** | GitHub issue/PR workflow |
| Run CI/CD actions | **gh-cli** | GitHub Actions management |
| Commit local changes | **git-commit-push** | Safe, approved commits |
| Push to remote | **git-commit-push** | User-approved push |
| Code review | **gh-cli** | PR review and comments |
| Release management | **gh-cli** | Version releases |

## Topics

- github-cli
- gh
- git
- ai-agent
- skill
- cli-reference
- automation
- workflow
- commit-guidelines

## Version

- **gh-cli skill**: Based on GitHub CLI version 2.85.0 (January 2026)
- **git-commit-push skill**: Best practices from real-world error cases

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

## License

This repository is provided as-is for use with AI agents. Refer to respective tool documentation for licensing details.

## Resources

- [Official GitHub CLI Documentation](https://cli.github.com/manual/)
- [GitHub CLI GitHub Repository](https://github.com/cli/cli)
- [GitHub CLI Releases](https://github.com/cli/cli/releases)
- [Git Documentation](https://git-scm.com/doc)
