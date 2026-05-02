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

### 1. Use `--body` with File Content Pattern for Comment Body

The `gh-pr-review` extension only supports `--body string`, **NOT `--body-file`** like native `gh` commands.

**NEVER pass `--body` with inline text containing special characters!** Backticks (`` ` ``) are escape characters in PowerShell and will cause garbled output or parsing errors.

```powershell
# Windows (PowerShell)
$body = Get-Content "workspace_dir/temp/review-body.md" -Raw
gh pr-review review 123 --add-comment --path "src/main.py" --line 42 `
  --body $body `
  --review-id "PRR_xxxxx"
```

```bash
# Linux / macOS (bash)
gh pr-review review 123 --add-comment --path "src/main.py" --line 42 \
  --body "$(cat workspace_dir/temp/review-body.md)" \
  --review-id "PRR_xxxxx"
```

```
# ❌ WRONG - inline body with special characters
gh pr-review review 123 --add-comment --body "Use `code()` function here..."
```

### 2. Export Review Output to Files

**NEVER view review output directly in terminal!** Output may be truncated. Always export to a local file first, then read it.

**⚠️ Redirection Warning:** `| Out-File` and `2>&1` may NOT work correctly with `gh` output. Only `>` redirection is reliable.

**TraeAI Terminal:** In the TraeAI agent terminal, `2>&1` merges stderr into stdout BEFORE `>` redirect, corrupting the output file. If you must capture stderr, use parentheses: `(gh pr-review review view ... > file) 2>&1`.

**JSON Output:** `gh pr-review review view` outputs single-line JSON. Pipe through `python -m json.tool` for readable formatting before writing to file.

```powershell
# ✅ CORRECT (ensure temp directory exists first)
mkdir workspace_dir/temp
gh pr-review review view 123 -R owner/repo | python -m json.tool > workspace_dir/temp/pr-123-reviews.md

# ❌ WRONG (may lose data)
gh pr-review review view 123 -R owner/repo

# ❌ WRONG: Out-File may produce incomplete output
gh pr-review review view 123 -R owner/repo | Out-File -FilePath pr-review.md

# ❌ WRONG: 2>&1 corrupts output in TraeAI terminal
gh pr-review review view 123 -R owner/repo > pr-review.md 2>&1

# ✅ If stderr capture is needed, use parentheses
(gh pr-review review view 123 -R owner/repo > pr-review.md) 2>&1
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
# Linux / macOS (bash)
# === Step 1: Start a pending review ===
gh pr-review review 123 --start -R owner/repo
# Output: {"reviewId":"PRR_lAbCdEf123456"}

# === Step 2: Add inline comments (repeat as needed) ===
gh pr-review review 123 \
  --add-comment --path "src/main.py" --line 42 --side RIGHT \
  --body "Potential null pointer risk here" \
  --review-id "PRR_lAbCdEf123456"

# === Step 3: Submit ===
gh pr-review review 123 --submit \
  --event REQUEST_CHANGES \
  --body "Please address the inline comments above" \
  --review-id "PRR_lAbCdEf123456"
```

```powershell
# Windows (PowerShell)
# === Step 1: Start a pending review ===
$START_OUTPUT = gh pr-review review 123 --start -R owner/repo | ConvertFrom-Json
$REVIEW_ID = $START_OUTPUT.reviewId

# === Step 2: Add inline comments (repeat as needed) ===
gh pr-review review 123 --add-comment `
  --path "src/main.py" --line 42 --side RIGHT `
  --body "Potential null pointer risk here" `
  --review-id $REVIEW_ID

# === Step 3: Submit ===
gh pr-review review 123 --submit `
  --event REQUEST_CHANGES `
  --body "Please address the inline comments above" `
  --review-id $REVIEW_ID
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
| `--body` | Comment body text (**PowerShell: `$body = Get-Content "file" -Raw` / bash: `$(cat file)`**) |
| `--commit` | Commit SHA (defaults to current HEAD) |
| `--review-id` | GraphQL review node ID from `--start` output |

---

## 2. Reply to Review Comments

Reply to existing review threads (inline comments from reviewers).

> **⚠️ Common mistake**: `gh pr reply` does NOT exist! Native `gh` has no `pr reply` subcommand. Always use the **gh-pr-review extension** for replying to review threads.

```bash
# Linux / macOS: List unresolved threads first to find thread-id
gh pr-review threads list 123 -R owner/repo --unresolved > workspace_dir/temp/threads.md

# Reply to a specific thread
gh pr-review comments reply 123 \
  -R owner/repo \
  --thread-id "RT_mNoPqRsTuvWxyz" \
  --body "$(cat workspace_dir/temp/reply-body.md)"
```

```powershell
# Windows: List unresolved threads first to find thread-id
gh pr-review threads list 123 -R owner/repo --unresolved > workspace_dir/temp/threads.md

# Reply to a specific thread
$body = Get-Content "workspace_dir/temp/reply-body.md" -Raw
gh pr-review comments reply 123 `
  -R owner/repo `
  --thread-id "RT_mNoPqRsTuvWxyz" `
  --body $body
```

### When to Reply (PR Author Workflow)

#### Scenario A: You pushed a fix → remind human reviewer

```powershell
# Windows (PowerShell)
$body = Get-Content "workspace_dir/temp/reply.md" -Raw
gh pr-review comments reply 123 -R owner/repo `
  --thread-id "RT_mNoPqRsTuvWxyz" `
  --body $body
```

```bash
# Linux / macOS (bash)
gh pr-review comments reply 123 -R owner/repo \
  --thread-id "RT_mNoPqRsTuvWxyz" \
  --body "$(cat workspace_dir/temp/reply.md)"
```

#### Scenario B: You choose NOT to fix → explain why

```powershell
# Windows (PowerShell)
$body = Get-Content "workspace_dir/temp/reply.md" -Raw
gh pr-review comments reply 123 -R owner/repo `
  --thread-id "RT_mNoPqRsTuvWxyz" `
  --body $body
```

```bash
# Linux / macOS (bash)
gh pr-review comments reply 123 -R owner/repo \
  --thread-id "RT_mNoPqRsTuvWxyz" \
  --body "$(cat workspace_dir/temp/reply.md)"
```

#### Scenario C: Reviewer is AI Bot → wait and check

```powershell
# Push fix, then wait 2 minutes for bot to auto-resolve (with progress)
120..1 | ForEach-Object { Write-Host "`r⏱️ 等待 AI Bot 审查... 剩余 $_ 秒" -NoNewline; Start-Sleep 1 }; Write-Host "`n✅ 2分钟已过，检查结果"

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

**⚠️ CRITICAL: Always export review output to local file for viewing**

**NEVER view review output directly in terminal!** Output may be truncated, causing loss of important information like bug reports, code suggestions, and error details.

**⚠️ Redirection Warning:** `| Out-File` and `2>&1` may NOT work correctly with `gh` output. Only `>` redirection is reliable.

**TraeAI Terminal:** In the TraeAI agent terminal, `2>&1` merges stderr into stdout BEFORE `>` redirect, corrupting the output file (e.g., only `--` in the file). If you must capture stderr, use parentheses: `(gh pr-review review view ... > file) 2>&1`.

**JSON Output:** `gh pr-review review view` outputs single-line JSON. Pipe through `python -m json.tool` for readable formatting before writing to file.

```bash
# Step 0: Ensure temp directory exists
mkdir -p workspace_dir/temp

# Step 1: Export review data to file
gh pr-review review view 123 -R owner/repo | python -m json.tool > workspace_dir/temp/pr-123-reviews.md

# Step 2: Read the exported file
cat workspace_dir/temp/pr-123-reviews.md
```

```powershell
# Windows (PowerShell)
mkdir workspace_dir/temp
gh pr-review review view 123 -R owner/repo | python -m json.tool > workspace_dir/temp/pr-123-reviews.md

# WRONG: Out-File may produce incomplete output
gh pr-review review view 123 -R owner/repo | Out-File -FilePath pr-review.md  # ❌

# WRONG: 2>&1 may produce empty output
gh pr-review review view 123 -R owner/repo > pr-review.md 2>&1  # ❌
```

### Filter Options

```bash
# Unresolved threads only, exclude outdated
gh pr-review review view 123 -R owner/repo --unresolved --not_outdated | python -m json.tool > workspace_dir/temp/pr-123-reviews.md

# Filter by specific reviewer
gh pr-review review view 123 -R owner/repo --reviewer alice | python -m json.tool > workspace_dir/temp/pr-123-reviews.md

# Filter by review state(s)
gh pr-review review view 123 -R owner/repo --states APPROVED,CHANGES_REQUESTED | python -m json.tool > workspace_dir/temp/pr-123-reviews.md

# Include comment_node_id (useful for scripting replies)
gh pr-review review view 123 -R owner/repo --include-comment-node-id | python -m json.tool > workspace_dir/temp/pr-123-reviews.md

# Limit replies per thread (reduce noise)
gh pr-review review view 123 -R owner/repo --tail 3 | python -m json.tool > workspace_dir/temp/pr-123-reviews.md
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
# Linux / macOS (bash)
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

```powershell
# Windows (PowerShell)
# 1. View PR diff first
gh pr diff 123 -R owner/repo > workspace_dir/temp/pr-diff.patch

# 2. Start pending review
$REVIEW_OUTPUT = gh pr-review review 123 --start -R owner/repo | ConvertFrom-Json
$REVIEW_ID = $REVIEW_OUTPUT.reviewId

# 3. Add inline comments while reading the diff
$body1 = Get-Content "workspace_dir/temp/comment1.md" -Raw
gh pr-review review 123 --add-comment `
  --path "src/auth.ts" --line 25 --side RIGHT `
  --body $body1 `
  --review-id $REVIEW_ID

# 4. Submit with verdict
$bodySummary = Get-Content "workspace_dir/temp/review-summary.md" -Raw
gh pr-review review 123 --submit `
  --event REQUEST_CHANGES `
  --body $bodySummary `
  --review-id $REVIEW_ID
```

### As PR Author: Fix & Respond Cycle

```bash
# Linux / macOS (bash)
# 1. Check what needs fixing
gh pr-review review view 123 -R owner/repo --unresolved --not_outdated > workspace_dir/temp/todo.md

# 2. Make changes, commit, push
git status
git add <explicit-file-paths>
git commit -m "Address review feedback" && git push

# 3. Decide response based on reviewer type:
#    - AI Bot: wait 2 min, then re-check threads
#    - Human: reply to thread with notification
#    - Or explain if not fixing
```

```powershell
# Windows (PowerShell)
# 1. Check what needs fixing
gh pr-review review view 123 -R owner/repo --unresolved --not_outdated > workspace_dir/temp/todo.md

# 2. Make changes, commit, push
git status
git add <explicit-file-paths>
git commit -m "Address review feedback"; git push

# 3. Decide response based on reviewer type:
#    - AI Bot: wait 2 min, then re-check threads
#    - Human: reply to thread with notification
#    - Or explain if not fixing
```

---

## See Also

- **[[references/prs.md|Pull Requests]]** - Native `gh pr` commands (create, list, merge, etc.)
- **[[references/advanced.md|API & Advanced]]** - Direct API calls, extensions management
