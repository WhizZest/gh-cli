---
name: "git-commit-push"
description: "Git提交与推送规范指南。执行git commit/push前必须调用，确保遵循用户审核、原子提交、精准文件选择等原则。"
---

# Git Commit & Push 规范指南

本skill总结了Git提交与推送的最佳实践、核心原则和工作流规范，所有操作都基于实际错误案例总结。

## Git Commit 核心原则

### 1. 用户审核原则（最重要）

```
❌ 禁止擅自执行 git commit 或 git push
✅ 提交前必须执行基本测试通过
✅ 每次提交前必须征询用户意见
✅ 让用户审核 commit message 内容
✅ 如果没有特别说明，用户的同意只针对一次操作，下次需重新询问
```

### 2 原子性提交
- 一个 commit 只做一件事（单一职责）
- 不要将多个不相关的修改混在一起
- 便于回滚、cherry-pick 和代码审查

### 3. 完整性
- 提交前确保代码可编译/可运行
- 不要提交半成品或调试代码
- 相关文件一次性提交（避免遗漏）

### 4. 及时性
- 完成一个小功能就提交
- 不要积累大量修改再一次性提交

### 5. 文件选择原则

```
❌ 禁止使用 git add -A 或 git add .
✅ 明确指定要提交的文件
✅ 提交前必须执行 git status 检查
✅ 临时文件应创建在`<workspace_dir>/temp/`目录下
✅ 提交前如果发现有临时文件，必须先将它们移动到 `<workspace_dir>/temp/`目录下
✅ 确认只有核心业务逻辑文件
```

## Git Push 核心原则

### 推送可运行的代码
- 不要推送编译失败或有明显 bug 的代码
- 确保代码通过基本测试
- 不要频繁推送微小修改，积累几个 commit 再统一推送

### 为什么不要“频繁 Push”？

1. 避免打扰队友
2. 保持代码质量
3. 便于回滚和合并
4. CI 资源浪费：每次 push 都会触发自动化测试、构建

## Git Commit 最佳实践

### 正确做法

**1. 明确指定文件**
```bash
# 方式1：明确指定文件
git add src/weread/catalog.ts src/weread/outline.ts

# 方式2：交互式添加
git add -p
```

**2. 提交前检查**
```bash
# 查看待提交文件
git status

# 如果发现有临时文件，先移动到 `<workspace_dir>/temp/` 目录下
mv temp-file.txt <workspace_dir>/temp/

# 查看暂存区差异
git diff --cached
```

**3. 规范的 Commit Message**
```
<type>(<scope>): <subject>

类型：
- feat: 新功能
- fix: 修复bug
- refactor: 重构
- docs: 文档
- style: 格式
- test: 测试
- chore: 构建/工具
```
**重要：** 如果Commit Message超过2行，必须在 `<workspace_dir>/temp/`目录下创建commit message临时文件，提交方式为 `git commit --file=<workspace_dir>/temp/git-commit-msg.txt` 。

### 常见错误

**1. 盲目使用 git add -A**
```bash
# ❌ 错误：添加所有文件（包括临时文件、文档等）
git add -A && git commit -m "..."

# ✅ 正确：明确指定文件
git add src/file1.ts src/file2.ts
```

**2. 临时文件混入提交**
```bash
# ❌ 临时文件创建在工作目录
.git-commit-msg.txt
.pr-description.md

# ✅ 使用<workspace_dir>/temp/`临时目录
<workspace_dir>/temp/git-commit-msg.txt
```

**3. 替换已创建PR的commit**
如果commit已被push，且该分支已创建PR，禁止直接amend。

### 分支合并策略（保持 PR 历史干净）

**场景：将主分支更新合入功能分支时**

**❌ 不推荐：使用 merge**
```bash
git merge master
# 问题：会产生额外的 "Merge branch 'master' into feature/xxx" commit
# PR 中会显示多余的合并提交，历史不够线性
```

**✅ 推荐：使用 rebase**
```bash
# 步骤1：更新本地主分支
git checkout master && git pull origin master

# 步骤2：切回功能分支并变基
git checkout feature/your-feature
git rebase master

# 步骤3：如果之前已经推送过，需要强制推送（必须先征询用户意见）
# ⚠️ force push 会重写远程历史，必须获得用户明确同意后才能执行
git push --force-with-lease origin feature/your-feature
```

**优势：**
- 保持线性提交历史，PR 只显示有意义的 commits
- 避免多余的 merge commit 污染历史图
- 符合大多数开源项目的贡献规范

**⚠️ 注意事项：**
- rebase 会重写提交历史，**仅在本地或个人分支上使用**
- 如果已推送到远程并被他人基于开发，禁止 rebase
- 公共分支（如 master/main）永远不要 rebase
- **强制推送（force push）必须征询用户意见**，不得擅自执行

## 撤销操作规范

### 1. 撤销暂存

```bash
# ✅ 撤销特定文件的暂存
git restore --staged <file>

# ✅ 撤销所有暂存
git restore --staged .
```

### 2. 撤销提交

```bash
# ✅ 撤销提交，保留修改
git reset --soft HEAD~1

# ⚠️ 撤销提交和暂存，保留工作区
git reset HEAD~1

# ❌ 危险：撤销并丢弃所有修改
git reset --hard HEAD~1
```

### 3. 从Git中移除文件

```bash
# ✅ 从Git中移除，保留本地文件
git rm --cached <file>

# ❌ 删除文件并从Git中移除
git rm <file>
```

## 检查清单

### 提交前

- [ ] 是否创建分支？
- [ ] 是否征询用户意见？
- [ ] 是否执行 git status 检查？
- [ ] 是否已将临时文件移动到 `<workspace_dir>/temp/`目录下？
- [ ] 是否明确指定文件？
- [ ] 是否创建commit message临时文件？
- [ ] Commit message 是否规范？
- [ ] 是否超出PR范围？

### 推送前

- [ ] 是否征询用户意见？
- [ ] 功能是否开发完成？
- [ ] 测试是否通过？
- [ ] 是否需要整理commit历史？

## 不要做的事 ❌

- 不要更新git config
- 不要在没有用户同意的情况下运行破坏性命令（如force reset）
- 不要在没有用户同意的情况下跳过hooks（如--no-verify）
- 不要强制推送到main/master分支
- 如果hooks失败，修复并创建新的commit（不要amend）
- 不要提交敏感信息（密码、密钥、Token）