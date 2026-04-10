# GitHub CLI Codespaces (gh codespace)

Complete reference for GitHub Codespaces management commands.

## List and View Codespaces

```bash
# List codespaces
gh codespace list

# View codespace details
gh codespace view
```

## Create Codespaces

```bash
# Create codespace
gh codespace create

# Create with specific repository
gh codespace create --repo owner/repo

# Create with branch
gh codespace create --branch develop

# Create with specific machine
gh codespace create --machine premiumLinux
```

## Connect to Codespaces

```bash
# SSH into codespace
gh codespace ssh

# SSH with specific command
gh codespace ssh --command "cd /workspaces && ls"

# Open codespace in browser
gh codespace code

# Open in VS Code
gh codespace code --codec

# Open with specific path
gh codespace code --path /workspaces/repo
```

## Manage Codespaces

```bash
# Stop codespace
gh codespace stop

# Delete codespace
gh codespace delete

# View logs
gh codespace logs --tail 100

# View ports
gh codespace ports

# Forward port
gh codespace cp 8080:8080

# Rebuild codespace
gh codespace rebuild

# Edit codespace
gh codespace edit --machine standardLinux

# Jupyter support
gh codespace jupyter
```

## File Operations

```bash
# Copy files to/from codespace
gh codespace cp file.txt :/workspaces/file.txt
gh codespace cp :/workspaces/file.txt ./file.txt
```

## See Also
- [Main SKILL.md](../SKILL.md) - Return to main documentation
