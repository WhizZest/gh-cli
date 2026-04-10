# GitHub CLI Issues (gh issue)

Complete reference for GitHub issue management commands.

## Create Issue

```bash
# Create issue interactively
gh issue create

# Create with title
gh issue create --title "Bug: Login not working"

# Create with title and body
gh issue create \
  --title "Bug: Login not working" \
  --body "Steps to reproduce..."

# Create with body from file
gh issue create --body-file issue.md

# Create with labels
gh issue create --title "Fix bug" --labels bug,high-priority

# Create with assignees
gh issue create --title "Fix bug" --assignee user1,user2

# Create in specific repository
gh issue create --repo owner/repo --title "Issue title"

# Create issue from web
gh issue create --web
```

## List Issues

```bash
# List all open issues
gh issue list

# List all issues (including closed)
gh issue list --state all

# List closed issues
gh issue list --state closed

# Limit results
gh issue list --limit 50

# Filter by assignee
gh issue list --assignee username
gh issue list --assignee @me

# Filter by labels
gh issue list --labels bug,enhancement

# Filter by milestone
gh issue list --milestone "v1.0"

# Search/filter
gh issue list --search "is:open is:issue label:bug"

# JSON output
gh issue list --json number,title,state,author

# Table view
gh issue list --json number,title,labels --jq '.[] | [.number, .title, .labels[].name] | @tsv'

# Show comments count
gh issue list --json number,title,comments --jq '.[] | [.number, .title, .comments]'

# Sort by
gh issue list --sort created --order desc
```

## View Issue

```bash
# View issue
gh issue view 123

# View with comments
gh issue view 123 --comments

# View in browser
gh issue view 123 --web

# JSON output
gh issue view 123 --json title,body,state,labels,comments

# View specific fields
gh issue view 123 --json title --jq '.title'
```

## Edit Issue

```bash
# Edit interactively
gh issue edit 123

# Edit title
gh issue edit 123 --title "New title"

# Edit body
gh issue edit 123 --body "New description"

# Add labels
gh issue edit 123 --add-label bug,high-priority

# Remove labels
gh issue edit 123 --remove-label stale

# Add assignees
gh issue edit 123 --add-assignee user1,user2

# Remove assignees
gh issue edit 123 --remove-assignee user1

# Set milestone
gh issue edit 123 --milestone "v1.0"
```

## Close/Reopen Issue

```bash
# Close issue
gh issue close 123

# Close with comment
gh issue close 123 --comment "Fixed in PR #456"

# Reopen issue
gh issue reopen 123
```

## Comment on Issue

### Best Practices for Issue Comments

**⚠️ Important: Use `--body-file` for Long or Complex Content**

When adding comments with long text, special characters, or formatting, use `--body-file` parameter to avoid CLI argument parsing errors:

```bash
# Recommended: Use file input for complex content
gh issue comment 123 --body-file comment.txt

# Edit last comment using file
gh issue comment 123 --edit-last --body-file updated-comment.txt

# Read from stdin
gh issue comment 123 --body-file -
```

**Common Issues and Solutions:**

1. **CLI Argument Parsing Error**
   - Error: `accepts 1 arg(s), received 8`
   - Cause: Long text with spaces, quotes, or special characters (especially backticks) in `--body` parameter causes escaping issues
   - Solution: Use `--body-file` parameter instead of `--body`

2. **Permission Errors**
   - Error: `your authentication token is missing required scopes`
   - Cause: GitHub CLI lacks necessary permissions for the operation
   - Solution: Refresh authentication with required scopes:
     ```bash
     gh auth status
     gh auth refresh -s read:project,workflow
     ```

3. **Editing Comments**
   - Use `--edit-last` to modify your most recent comment
   - Combine with `--body-file` for complex updates
   - Use `--delete-last` to remove your last comment

**Example Workflow:**

```bash
# 1. Create comment file
cat > comment.txt << 'EOF'
Hi @username,

Here's a detailed explanation with:
- Multiple bullet points
- Code examples
- Links to resources

EOF

# 2. Post comment using file
gh issue comment 123 --body-file comment.txt

# 3. Update if needed
cat > updated-comment.txt << 'EOF'
Hi @username,

Updated content here...

EOF

gh issue comment 123 --edit-last --body-file updated-comment.txt
```

### Comment Commands

```bash
# Add comment
gh issue comment 123 --body "This looks good!"

# Edit comment
gh issue comment 123 --edit 456789 --body "Updated comment"

# Delete comment
gh issue comment 123 --delete 456789
```

## Issue Status

```bash
# Show issue status summary
gh issue status

# Status for specific repository
gh issue status --repo owner/repo
```

## Pin/Unpin Issues

```bash
# Pin issue (pinned to repo dashboard)
gh issue pin 123

# Unpin issue
gh issue unpin 123
```

## Lock/Unlock Issue

```bash
# Lock conversation
gh issue lock 123

# Lock with reason
gh issue lock 123 --reason off-topic

# Unlock
gh issue unlock 123
```

## Transfer Issue

```bash
# Transfer to another repository
gh issue transfer 123 --repo owner/new-repo
```

## Delete Issue

```bash
# Delete issue
gh issue delete 123

# Confirm without prompt
gh issue delete 123 --yes
```

## Develop Issue (Draft PR)

```bash
# Create draft PR from issue
gh issue develop 123

# Create in specific branch
gh issue develop 123 --branch fix/issue-123

# Create with base branch
gh issue develop 123 --base main
```

## See Also
- [Main SKILL.md](../SKILL.md) - Return to main documentation
- [Pull Requests](./prs.md) - Pull request workflows
- [Labels](./labels.md) - Label management
