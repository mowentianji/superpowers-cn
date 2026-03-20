---
name: writing-plans
description: 当一个多步骤任务在接触代码前已经有设计、规范或明确要求时使用
---

# 编写实施计划

## 概述

请假设执行这份计划的工程师对当前代码库几乎没有背景知识，也不熟悉问题领域，甚至不一定擅长测试设计。你的工作是为他们写出**完整、可执行、可交接**的实施计划：明确每个任务要改哪些文件、写哪些代码、补哪些测试、参考哪些文档、如何验证结果。

把整个实现拆成一组足够小、足够清晰的任务。坚持：**DRY、YAGNI、TDD、频繁提交。**

**开始时必须明确说明：**“我正在使用 writing-plans skill 来创建实施计划。”

**上下文：** 这项工作应该在专用 worktree 中进行（通常由 `brainstorming` skill 的流程创建）。

**计划保存路径：** `docs/plans/YYYY-MM-DD-<feature-name>.md`

## 子任务粒度

**每一步都应该是一个清晰动作（通常 2-5 分钟）：**
- “编写一个失败的测试” —— 一个步骤
- “运行测试确认它失败” —— 一个步骤
- “写出让测试通过的最小实现” —— 一个步骤
- “再次运行测试确认通过” —— 一个步骤
- “提交本次变更” —— 一个步骤

## 计划文档标题模板

**每份计划都必须以下面这个标题块开头：**

```markdown
# [功能名称] 实施计划

> **给 Claude：** 必须使用 `superpowers:executing-plans` 子技能，按任务逐项执行本计划。

**目标：** [一句话说明本次要构建什么]

**架构方案：** [用 2-3 句话概述实现思路]

**技术栈：** [关键技术 / 框架 / 库]

---
```

## 任务结构模板

````markdown
### 任务 N：[组件或子功能名称]

**涉及文件：**
- 新建：`exact/path/to/file.py`
- 修改：`exact/path/to/existing.py:123-145`
- 测试：`tests/exact/path/to/test.py`

**步骤 1：先写一个失败的测试**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

**步骤 2：运行测试，确认它先失败**

运行：`pytest tests/path/test.py::test_name -v`
预期：**失败（FAIL）**，并出现类似“function not defined”的报错

**步骤 3：编写最小实现，让测试通过**

```python
def function(input):
    return expected
```

**步骤 4：再次运行测试，确认它通过**

运行：`pytest tests/path/test.py::test_name -v`
预期：**通过（PASS）**

**步骤 5：提交变更**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific behavior"
```
````

## 编写要求

- 始终写出**精确文件路径**
- 在计划里给出**尽量完整的代码或伪代码骨架**，不要只写“补充校验逻辑”这种抽象描述
- 所有命令都写明，并注明预期结果
- 使用 `@` 语法引用相关 skill 或文档
- 全程贯彻：**DRY、YAGNI、TDD、频繁提交**

## 输出文档语言要求

- 计划文档正文优先使用中文
- 建议统一采用中文章节名，例如：
  - `目标`
  - `架构方案`
  - `技术栈`
  - `任务拆解`
  - `验证方式`
  - `风险与注意事项`
- 文件路径和文件名约定可以保留英文，以维持仓库兼容性

## 执行交接

保存计划后，必须向用户提供两种执行方式：

**“实施计划已完成，并已保存到 `docs/plans/<filename>.md`。接下来有两种执行方式：**

**1. 子代理驱动（当前会话内）** —— 我在当前会话里为每个任务派发新的子代理，并在任务之间进行审查，快速迭代。

**2. 并行会话（单独执行）** —— 你在新的会话中使用 `executing-plans` 按检查点批量执行。

**你希望采用哪一种方式？**”

**如果用户选择“子代理驱动”：**
- **必须使用的子技能：** `superpowers:subagent-driven-development`
- 保持在当前会话内执行
- 每个任务用新的子代理实现，并在任务后进行代码审查

**如果用户选择“并行会话”：**
- 指导用户在 worktree 中开启一个新会话
- **必须使用的子技能：** 在新会话中使用 `superpowers:executing-plans`
