---
name: gh-cli
description: GitHub CLI (gh) comprehensive reference for repositories, issues, pull requests, Actions, projects, releases, gists, codespaces, organizations, extensions, and all GitHub operations from the command line.
---

# GitHub CLI (gh) - 导航索引

## ⚠️ 强制规则

**执行任何 gh 操作前，必须先阅读对应的参考文档。** 本文档只提供导航，不包含操作细节。跳过参考文档直接操作是违反规范的。

## 参考文档索引

按操作场景查找对应的参考文档：

### 认证与配置

| 文档 | 适用场景 |
|------|----------|
| [[references/auth.md\|Authentication]] | 登录、token 管理、账号切换 |
| [[references/config.md\|Configuration]] | 环境配置、输出格式、最佳实践 |

### 仓库与代码

| 文档 | 适用场景 |
|------|----------|
| [[references/repo.md\|Repositories]] | 创建、克隆、fork、编辑、删除仓库 |
| [[references/search.md\|Search]] | 搜索代码、提交、issue、PR、仓库 |

### Issue 管理

| 文档 | 适用场景 |
|------|----------|
| [[references/issues.md\|Issues]] | 创建、列出、评论、管理 issue |

### Pull Request

| 文档 | 适用场景 |
|------|----------|
| [[references/prs.md\|Pull Requests]] | PR 创建、查看、合并、分支清理 |
| [[references/pr-reviews.md\|PR Reviews]] | 行内代码审查、review 线程、gh-pr-review 扩展 |

### CI/CD 与自动化

| 文档 | 适用场景 |
|------|----------|
| [[references/actions.md\|GitHub Actions]] | 工作流、运行、secrets、变量 |

### 项目管理

| 文档 | 适用场景 |
|------|----------|
| [[references/projects.md\|Projects]] | GitHub Projects 管理 |
| [[references/releases.md\|Releases]] | 版本发布与资产管理 |

### 开发工具

| 文档 | 适用场景 |
|------|----------|
| [[references/gists.md\|Gists]] | Gist 创建与管理 |
| [[references/codespaces.md\|Codespaces]] | 远程开发环境 |

### 高级功能

| 文档 | 适用场景 |
|------|----------|
| [[references/advanced.md\|API & Advanced]] | API 调用、扩展、别名、标签、SSH/GPG 密钥 |

## 安装与验证

```bash
# Windows
winget install --id GitHub.cli

# macOS
brew install gh

# Linux
# 参考 https://github.com/cli/cli#installation

# 验证安装
gh --version
```
