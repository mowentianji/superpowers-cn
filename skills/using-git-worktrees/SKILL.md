---
name: using-git-worktrees
description: 在开始需要与当前工作空间隔离的功能工作时或在执行实施计划之前使用 - 通过智能目录选择和安全验证创建隔离的 git 工作树
---

# 使用 Git 工作树

## 概述

Git 工作树创建共享同一存储库的独立工作区，允许同时在多个分支上工作而无需切换。

**核心原则：**系统的目录选择+安全验证=可靠隔离。

**开始时宣布：**“我正在使用 using-git-worktrees 技能来设置一个隔离的工作区。”

## 目录选择过程

请遵循以下优先顺序：

### 1. 检查现有目录

```bash
# Check in priority order
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
ls -d worktrees 2>/dev/null      # Alternative
```

**如果找到：** 使用该目录。如果两者都存在，则 `.worktrees` 获胜。

### 2. 检查 CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**如果指定偏好：** 无需询问即可使用。

### 3. 询问用户

如果不存在目录并且没有 CLAUDE.md 首选项：

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. ~/.config/superpowers/worktrees/<project-name>/ (global location)

Which would you prefer?
```

## 安全验证

### 对于项目本地目录（.worktrees 或 worktrees）

**在创建工作树之前必须验证目录是否被忽略：**

```bash
# Check if directory is ignored (respects local, global, and system gitignore)
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果不忽略：**

根据杰西的规则“立即修复损坏的东西”：
1. 将适当的行添加到 .gitignore
2. 提交更改
3. 继续创建工作树

**为什么重要：** 防止意外地将工作树内容提交到存储库。

### 对于全局目录（~/.config/superpowers/worktrees）

无需 .gitignore 验证 - 完全在项目之外。

## 创作步骤

### 1. 检测项目名称

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. 创建工作树

```bash
# Determine full path
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# Create worktree with new branch
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. 运行项目设置

自动检测并运行适当的设置：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. 验证清洁基线

运行测试以确保工作树干净启动：

```bash
# Examples - use project-appropriate command
npm test
cargo test
pytest
go test ./...
```

**如果测试失败：** 报告失败，询问是否继续或调查。

**如果测试通过：** 报告准备就绪。

### 5. 举报地点

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## 快速参考

| 情况 | 行动 |
|-----------|--------|
| `.worktrees/` 存在 | 使用它（验证忽略） |
| `worktrees/` 存在 | 使用它（验证忽略） |
| 两者都存在 | 使用 `.worktrees/` |
| 两者都不存在 | 检查 CLAUDE.md → 询问用户 |
| 目录未被忽略 | 添加到 .gitignore + 提交 |
| 测试在基线期间失败 | 报告失败+询问 |
| 没有 package.json/Cargo.toml | 跳过依赖安装 |

## 常见错误

### 跳过忽略验证

- **问题：** 工作树内容被跟踪，污染 git 状态
- **修复：** 在创建项目本地工作树之前始终使用 `git check-ignore`

### 假设目录位置

- **问题：** 造成不一致，违反项目约定
- **修复：**遵循优先级：现有> CLAUDE.md>询问

### 继续失败的测试

- **问题：** 无法区分新错误和先前存在的问题
- **修复：** 报告失败，获得明确的许可才能继续

### 硬编码设置命令

- **问题：** 使用不同工具的项目中断
- **修复：** 从项目文件（package.json 等）自动检测

## 示例工作流程

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.

[Check .worktrees/ - exists]
[Verify ignored - git check-ignore confirms .worktrees/ is ignored]
[Create worktree: git worktree add .worktrees/auth -b feature/auth]
[Run npm install]
[Run npm test - 47 passing]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## 危险信号

**绝不：**
- 创建工作树而不验证它是否被忽略（项目本地）
- 跳过基线测试验证
- 不询问就继续失败的测试
- 当目录位置不明确时假定目录位置
- 跳过 CLAUDE.md 检查

**总是：**
- 遵循目录优先级：existing>CLAUDE.md>ask
- 验证项目本地的目录被忽略
- 自动检测并运行项目设置
- 验证干净的测试基线

## 一体化

**调用者：**
- **头脑风暴**（第 4 阶段）- 当设计获得批准并随后实施时需要
- **子代理驱动开发** - 执行任何任务之前需要
- **执行计划** - 执行任何任务之前需要
- 任何需要独立工作空间的技能

**搭配：**
- **完成开发分支** - 需要在工作完成后进行清理
