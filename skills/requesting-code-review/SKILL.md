---
name: requesting-code-review
description: 在完成任务、实现关键功能或合并前使用，以验证工作是否满足要求
---

# 请求代码审查

派发 `superpowers:code-reviewer` 子代理，在问题连锁放大之前尽早发现风险。

**核心原则：** 尽早审、经常审。

## 何时请求审查

**强制场景：**
- 在子代理驱动开发中，每个任务完成后都要审查
- 完成主要功能后要审查
- 合并到主分支之前要审查

**可选但很有价值：**
- 当你卡住时（引入新视角）
- 重大重构之前（先做基线检查）
- 修复复杂 Bug 之后

## 如何请求

**1. 获取 Git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派发代码审查子代理：**

使用 `superpowers:code-reviewer` 类型的 Task 工具，并套用 `code-reviewer.md` 模板。

**需要替换的占位符：**
- `{WHAT_WAS_IMPLEMENTED}` —— 本次刚实现的内容
- `{PLAN_OR_REQUIREMENTS}` —— 计划或需求说明
- `{BASE_SHA}` —— 起始提交
- `{HEAD_SHA}` —— 当前提交
- `{DESCRIPTION}` —— 简短摘要

**3. 根据反馈采取行动：**
- 严重问题：立刻修复
- 重要问题：继续前必须修复
- 次要问题：记录下来，安排后续处理
- 如果你认为审查意见有误，可以提出技术性反驳，但要给出依据

## 示例

```text
[刚完成任务 2：新增校验功能]

你：我先做一次代码审查，再继续后面的任务。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派发 superpowers:code-reviewer 子代理]
  WHAT_WAS_IMPLEMENTED: 为会话索引新增校验与修复函数
  PLAN_OR_REQUIREMENTS: docs/plans/deployment-plan.md 中的任务 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: 新增 verifyIndex() 和 repairIndex()，支持 4 种问题类型

[子代理返回：]
  优点：架构清晰，测试真实有效
  问题：
    重要：缺少进度提示
    次要：用于上报间隔的魔法数字 100
  评估：修复后可继续推进

你：[补上进度提示]
[继续任务 3]
```

## 与工作流集成

**子代理驱动开发：**
- 每个任务后都审查
- 在问题变复杂之前提早发现
- 修复后再进入下一个任务

**执行计划：**
- 每个批次（例如 3 个任务）后审查一次
- 获取反馈、应用反馈、继续执行

**临时开发：**
- 合并前做审查
- 卡住时做审查

## 危险信号

**绝对不要：**
- 因为“这个很简单”就跳过审查
- 忽略严重问题
- 带着未解决的重要问题继续往下做
- 无依据地驳回有效的技术反馈

**如果审查者判断有误：**
- 用技术推理回应
- 给出代码或测试作为证据
- 必要时要求对方进一步说明

请参阅模板：`requesting-code-review/code-reviewer.md`
