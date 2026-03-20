---
name: using-superpowers
description: 开始任何对话时使用 - 确定如何查找和使用技能，要求在任何响应（包括澄清问题）之前调用技能工具
---

<EXTREMELY-IMPORTANT>
如果您认为某项技能有 1% 的机会适用于您正在做的事情，那么您绝对必须调用该技能。

如果某项技能适用于您的任务，您就别无选择。你必须使用它。

这是没有商量余地的。这不是可选的。你无法合理化自己的出路。
</EXTREMELY-IMPORTANT>

## 如何获取技能

**在克劳德代码中：** 使用 `Skill` 工具。当您调用一项技能时，其内容将被加载并呈现给您 - 直接遵循它。切勿对技能文件使用读取工具。

**在其他环境中：** 检查您的平台文档以了解如何加载技能。

# 使用技巧

## 规则

**在任何响应或操作之前调用相关或请求的技能。** 即使某项技能适用的可能性只有 1%，也意味着您应该调用该技能进行检查。如果调用的技能被证明不适合当前情况，则您不需要使用它。

```dot
digraph skill_flow {
    "User message received" [shape=doublecircle];
    "About to EnterPlanMode?" [shape=doublecircle];
    "Already brainstormed?" [shape=diamond];
    "Invoke brainstorming skill" [shape=box];
    "Might any skill apply?" [shape=diamond];
    "Invoke Skill tool" [shape=box];
    "Announce: 'Using [skill] to [purpose]'" [shape=box];
    "Has checklist?" [shape=diamond];
    "Create TodoWrite todo per item" [shape=box];
    "Follow skill exactly" [shape=box];
    "Respond (including clarifications)" [shape=doublecircle];

    "About to EnterPlanMode?" -> "Already brainstormed?";
    "Already brainstormed?" -> "Invoke brainstorming skill" [label="no"];
    "Already brainstormed?" -> "Might any skill apply?" [label="yes"];
    "Invoke brainstorming skill" -> "Might any skill apply?";

    "User message received" -> "Might any skill apply?";
    "Might any skill apply?" -> "Invoke Skill tool" [label="yes, even 1%"];
    "Might any skill apply?" -> "Respond (including clarifications)" [label="definitely not"];
    "Invoke Skill tool" -> "Announce: 'Using [skill] to [purpose]'";
    "Announce: 'Using [skill] to [purpose]'" -> "Has checklist?";
    "Has checklist?" -> "Create TodoWrite todo per item" [label="yes"];
    "Has checklist?" -> "Follow skill exactly" [label="no"];
    "Create TodoWrite todo per item" -> "Follow skill exactly";
}
```

## 危险信号

这些想法意味着停止——你正在合理化：

|思想|现实|
|---------|---------|
| “这只是一个简单的问题”|问题就是任务。检查技能。 |
| “我首先需要更多背景信息” |技能检查先于澄清问题。 |
| “让我先探索一下代码库”|技能告诉您如何探索。先检查一下。 |
| “我可以快速检查 git/files”|文件缺乏对话上下文。检查技能。 |
| “我先收集一下信息”|技能告诉您如何收集信息。 |
| “这不需要正式的技能”|如果存在技能，就使用它。 |
| “我记得这个技能”|技能不断发展。阅读当前版本。 |
| “这不算是任务”|行动=任务。检查技能。 |
| “技能太过分了”|简单的事情变得复杂。使用它。 |
| “我先做一件事” |做任何事情之前先检查一下。 |
| “这感觉很有成效” |无纪律的行动会浪费时间。技能可以防止这种情况发生。 |
| “我知道这意味着什么”|了解概念≠使用技能。调用它。 |

## 技能优先

当可以应用多种技能时，请使用以下顺序：

1. **流程技能优先**（头脑风暴、调试）-这些决定如何处理任务
2. **实现技巧第二**（frontend-design、mcp-builder）——这些指导执行

“让我们构建X”→首先集思广益，然后是实施技巧。
“修复此错误”→ 首先调试，然后再进行特定领域的技能。

## 技能类型

**严格**（TDD、调试）：严格遵循。不要适应纪律。

**灵活**（模式）：根据具体情况调整原则。

技能本身会告诉你哪个。

## 用户说明

说明说的是“做什么”，而不是“如何做”。 “添加 X”或“修复 Y”并不意味着跳过工作流程。
