# GitHub CLI PR Review Extension (gh-pr-review)

Complete reference for the `gh-pr-review` extension - inline code review comments, review thread management, and structured review views.

**Extension:** [agynio/gh-pr-review](https://github.com/agynio/gh-pr-review)

## Prerequisites

- **gh CLI** installed and authenticated (`gh auth login`)
- **Write access** to the target repository
- **Extension installed** (see Installation below)

### Installation / Update

```bash
# Install
gh extension install agynio/gh-pr-review

# Update to latest version
gh extension upgrade gh-pr-review

# Verify installation
gh pr-review --help
```

---

## Critical Best Practices

### 1. Use `--body "$(cat file)"` Pattern for Comment Body

The `gh-pr-review` extension only supports `--body string`, **NOT `--body-file`** like native `gh` commands.

**NEVER pass `--body` with inline text containing special characters!** Backticks (`` ` ``) are escape characters in PowerShell and will cause garbled output or parsing errors.

```powershell
# Step 1: Use the agent's built-in Write tool to create a temp file
# (Do NOT use shell commands - backticks will be mangled by PowerShell)
# workspace_dir is the root directory of the project
# File content: Your comment with `backticks` and special chars

# Step 2: Use command substitution to read file content
gh pr-review review 123 --add-comment --path "src/main.py" --line 42 `
  --body "$(cat workspace_dir/temp/review-body.md)" `
  --review-id "PRR_xxxxx"

# ❌ WRONG - inline body with special characters
gh pr-review review 123 --add-comment --body "Use `code()` function here..."
```

### 2. Export Review Output to Files

**NEVER view review output directly in terminal!** Output may be truncated.

```powershell
# ✅ CORRECT
gh pr-review review view 123 -R owner/repo > workspace_dir/temp/pr-review-123.md

# ❌ WRONG (may lose data)
gh pr-review review view 123 -R owner/repo
```

---

## Review Workflow Conventions

### Who Resolves Review Threads

| Role | Resolve Thread | Reply to Thread |
|------|---------------|-----------------|
| **PR Author (you)** | ❌ Usually NO | ✅ After pushing fixes, or explaining why not fixing |
| **Reviewer** | ✅ YES - after confirming fix | ✅ Initial review comments |

**Rule:** PR author should commit, push, and let the reviewer confirm and resolve. The PR author should NOT self-resolve unless explicitly appropriate.

### After Pushing a Fix - Decision Matrix

| Reviewer Type | Action | Details |
|--------------|--------|---------|
| **AI Bot** | Wait ~2 minutes, then check | AI bots typically auto-resolve within 2 minutes after a fix push |
| **Human** | Reply to remind | Push fix → reply to thread asking reviewer to re-check |
| **Any** | Reply with explanation | If choosing NOT to fix, reply with reasoning |

---

## 1. Inline Code Review

Native `gh pr review` only supports **overall** review status (approve/request-changes/comment). For **line-specific** code review comments, use `gh-pr-review`.

### Pending Review Workflow (3 Steps)

```bash
# === Step 1: Start a pending review ===
# This creates a draft review and returns a reviewId
gh pr-review review 123 --start -R owner/repo
# Output: {"reviewId":"PRR_lAbCdEf123456"}
# SAVE this reviewId for subsequent commands!

# === Step 2: Add inline comments (repeat as needed) ===

# Single-line comment on new code (RIGHT side)
gh pr-review review 123 \
  --add-comment \
  --path "src/main.py" \
  --line 42 \
  --side RIGHT \
  --body "Potential null pointer risk here" \
  --review-id "PRR_lAbCdEf123456"

# Multi-line comment (spanning lines 10-20)
gh pr-review review 123 \
  --add-comment \
  --path "src/utils.ts" \
  --start-line 10 --start-side RIGHT \
  --line 20 --side RIGHT \
  --body "This entire function lacks error handling" \
  --review-id "PRR_lAbCdEf123456"

# Comment on removed code (LEFT side)
gh pr-review review 123 \
  --add-comment \
  --path "src/legacy.js" \
  --line 15 \
  --side LEFT \
  --body "Why was this check removed?" \
  --review-id "PRR_lAbCdEf123456"

# === Step 3: Submit the review ===
gh pr-review review 123 \
  --submit \
  --event REQUEST_CHANGES \
  --body "Please address the inline comments above" \
  --review-id "PRR_lAbCdEf123456"
```

### Review Event Types

| Event | Flag | Meaning |
|-------|------|---------|
| Approve | `--event APPROVE` | LGTM, ready to merge |
| Request Changes | `--event REQUEST_CHANGES` | Fixes needed before merge |
| Comment Only | `--event COMMENT` (default) | Observations, no blocking opinion |

### Key Parameters Reference

| Parameter | Description |
|-----------|-------------|
| `--start` | Open a pending review, returns `reviewId` |
| `--add-comment` | Add an inline comment to pending review |
| `--submit` | Submit the pending review with final verdict |
| `--path` | File path relative to repository root |
| `--line` | Target line number |
| `--start-line` | Start line for multi-line range comment |
| `--side` | Diff side: `RIGHT` (new code) or `LEFT` (old code) |
| `--start-side` | Start side for multi-line range comment |
| `--event` | `APPROVE`, `REQUEST_CHANGES`, or `COMMENT` |
| `--body` | Comment body text (**use `$(cat file)` pattern!**) |
| `--commit` | Commit SHA (defaults to current HEAD) |
| `--review-id` | GraphQL review node ID from `--start` output |

---

## 2. Reply to Review Comments

Reply to existing review threads (inline comments from reviewers).

```bash
# List unresolved threads first to find thread-id
gh pr-review threads list 123 -R owner/repo --unresolved > workspace_dir/temp/threads.md

# Reply to a specific thread
gh pr-review comments reply 123 \
  -R owner/repo \
  --thread-id "RT_mNoPqRsTuvWxyz" \
  --body "$(cat workspace_dir/temp/reply-body.md)"
```

### When to Reply (PR Author Workflow)

#### Scenario A: You pushed a fix → remind human reviewer

```powershell
# Step 1: Use Write tool to create workspace_dir/temp/reply.md
# File content: Fix pushed in abc1234 - please re-check

# Step 2: Reply to notify
gh pr-review comments reply 123 -R owner/repo `
  --thread-id "RT_mNoPqRsTuvWxyz" `
  --body "$(cat workspace_dir/temp/reply.md)"
```

#### Scenario B: You choose NOT to fix → explain why

```powershell
# Step 1: Use Write tool to create workspace_dir/temp/reply.md
# File content: This is intentional - the null case is handled upstream by the caller

# Step 2: Reply with explanation
gh pr-review comments reply 123 -R owner/repo `
  --thread-id "RT_mNoPqRsTuvWxyz" `
  --body "$(cat workspace_dir/temp/reply.md)"
```

#### Scenario C: Reviewer is AI Bot → wait and check

```powershell
# Push fix, then wait 2 minutes for bot to auto-resolve
Start-Sleep -Seconds 120

# Check if thread is resolved
gh pr-review threads list 123 -R owner/repo --unresolved
```

### Reply Inside a Pending Review

When you have an active pending review (from `--start`), you can also reply within it:

```bash
gh pr-review comments reply 123 \
  --thread-id "RT_mNoPqRsTuvWxyz" \
  --body "Acknowledged, adding retry logic" \
  --review-id "PRR_lAbCdEf123456"
```

---

## 3. Manage Review Threads

### List Review Threads

```bash
# All threads
gh pr-review threads list 123 -R owner/repo > workspace_dir/temp/threads.md

# Unresolved only
gh pr-review threads list 123 -R owner/repo --unresolved

# Threads involving you (authored or resolvable)
gh pr-review threads list 123 -R owner/repo --mine

# Exclude outdated (code has changed since comment)
gh pr-review threads list 123 -R owner/repo # combine with review view --not_outdated
```

### Resolve / Unresolve Threads

```bash
# Resolve a thread (reviewer confirms fix is good)
gh pr-review threads resolve 123 \
  -R owner/repo \
  --thread-id "RT_mNoPqRsTuvWxyz"

# Reopen a resolved thread
gh pr-review threads unresolve 123 \
  -R owner/repo \
  --thread-id "RT_mNoPqRsTuvWxyz"
```

> ⚠️ **Convention:** As PR author, avoid self-resolving. Let the original reviewer resolve after confirming your fix.

---

## 4. Structured Review View

Get a comprehensive, machine-readable overview of all reviews on a PR.

```bash
# Full review summary (export to file!)
gh pr-review review view 123 -R owner/repo > workspace_dir/temp/pr-review-123.md

# Unresolved threads only, exclude outdated
gh pr-review review view 123 -R owner/repo --unresolved --not_outdated > workspace_dir/temp/pr-review-open.md

# Filter by specific reviewer
gh pr-review review view 123 -R owner/repo --reviewer alice

# Filter by review state(s)
gh pr-review review view 123 -R owner/repo --states APPROVED,CHANGES_REQUESTED

# Include comment_node_id (useful for scripting replies)
gh pr-review review view 123 -R owner/repo --include-comment-node-id

# Limit replies per thread (reduce noise)
gh pr-review review view 123 -R owner/repo --tail 3
```

### View Parameters Reference

| Parameter | Description |
|-----------|-------------|
| `--unresolved` | Show only unresolved threads |
| `--not_outdated` | Exclude threads where code has changed since comment |
| `--reviewer` | Filter to a specific reviewer's login |
| `--states` | Comma-separated: `APPROVED`, `CHANGES_REQUESTED`, `COMMENTED`, `DISMISSED` |
| `--include-comment-node-id` | Include GraphQL node IDs for comments (enables scripted replies) |
| `--tail N` | Show last N replies per thread (0 = all) |

---

## Complete Workflow Example

### As a Reviewer: Full Code Review Cycle

```bash
# 1. View PR diff first
gh pr diff 123 -R owner/repo > workspace_dir/temp/pr-diff.patch

# 2. Start pending review
REVIEW_OUTPUT=$(gh pr-review review 123 --start -R owner/repo)
REVIEW_ID=$(echo $REVIEW_OUTPUT | jq -r '.reviewId')

# 3. Add inline comments while reading the diff
gh pr-review review 123 --add-comment \
  --path "src/auth.ts" --line 25 --side RIGHT \
  --body "$(cat workspace_dir/temp/comment1.md)" \
  --review-id "$REVIEW_ID"

# 4. Submit with verdict
gh pr-review review 123 --submit \
  --event REQUEST_CHANGES \
  --body "$(cat workspace_dir/temp/review-summary.md)" \
  --review-id "$REVIEW_ID"
```

### As PR Author: Fix & Respond Cycle

```bash
# 1. Check what needs fixing
gh pr-review review view 123 -R owner/repo --unresolved --not_outdated > workspace_dir/temp/todo.md

# 2. Make changes, commit, push
git add . && git commit -m "Address review feedback" && git push

# 3. Decide response based on reviewer type:
#    - AI Bot: wait 2 min, then re-check threads
#    - Human: reply to thread with notification
#    - Or explain if not fixing
```

---

## See Also

- **[Main SKILL.md](../SKILL.md)** - Return to main documentation
- **[Pull Requests](./prs.md)** - Native `gh pr` commands (create, list, merge, etc.)
- **[API & Advanced](./advanced.md)** - Direct API calls, extensions management
