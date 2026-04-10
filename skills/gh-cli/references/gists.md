# GitHub CLI Gists (gh gist)

Complete reference for GitHub Gists management commands.

## List and View Gists

```bash
# List gists
gh gist list

# List all gists (including private)
gh gist list --public

# Limit results
gh gist list --limit 20

# View gist
gh gist view abc123

# View gist files
gh gist view abc123 --files
```

## Create Gists

```bash
# Create gist
gh gist create script.py

# Create gist with description
gh gist create script.py --desc "My script"

# Create public gist
gh gist create script.py --public

# Create multi-file gist
gh gist create file1.py file2.py

# Create from stdin
echo "print('hello')" | gh gist create
```

## Edit and Manage Gists

```bash
# Edit gist
gh gist edit abc123

# Delete gist
gh gist delete abc123

# Rename gist file
gh gist rename abc123 --filename old.py new.py
```

## Clone Gists

```bash
# Clone gist
gh gist clone abc123

# Clone to directory
gh gist clone abc123 my-directory
```

## See Also
- [Main SKILL.md](../SKILL.md) - Return to main documentation
