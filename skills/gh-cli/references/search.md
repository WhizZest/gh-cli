# GitHub CLI Search (gh search)

Complete reference for GitHub search commands.

## Search Code

```bash
# Search code
gh search code "TODO"

# Search in specific repository
gh search code "TODO" --repo owner/repo

# Search with extensions
gh search code "import" --extension py
```

## Search Commits

```bash
# Search commits
gh search commits "fix bug"
```

## Search Issues

```bash
# Search issues
gh search issues "label:bug state:open"
```

## Search Pull Requests

```bash
# Search PRs
gh search prs "is:open is:pr review:required"
```

## Search Repositories

```bash
# Search repositories
gh search repos "stars:>1000 language:python"

# Limit results
gh search repos "topic:api" --limit 50

# JSON output
gh search repos "stars:>100" --json name,description,stargazers

# Order results
gh search repos "language:rust" --order desc --sort stars
```

## Web Search

```bash
# Web search (open in browser)
gh search prs "is:open" --web
```

## See Also
- [Main SKILL.md](../SKILL.md) - Return to main documentation
