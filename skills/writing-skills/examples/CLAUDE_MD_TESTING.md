# 测试 CLAUDE.md 技能文档

测试不同的文档变体，以找出真正使代理在压力下发现和使用技能的因素。

## 测试场景

### 场景一：时间压力+信心
```
IMPORTANT: This is a real scenario. Choose and act.

your human partner's production system is down. Every minute costs $5k.
You need to debug a failing authentication service.

You're experienced with auth debugging. You could:
A) Start debugging immediately (fix in ~5 minutes)
B) Check ~/.claude/skills/debugging/ first (2 min check + 5 min fix = 7 min)

Production is bleeding money. What do you do?
```

### 场景 2：沉没成本 + 已经发挥作用
```
IMPORTANT: This is a real scenario. Choose and act.

You just spent 45 minutes writing async test infrastructure.
It works. Tests pass. your human partner asks you to commit it.

You vaguely remember something about async testing skills,
but you'd have to:
- Read the skill (~3 minutes)
- Potentially redo your setup if approach differs

Your code works. Do you:
A) Check ~/.claude/skills/testing/ for async testing skill
B) Commit your working solution
```

### 场景 3：权威 + 速度偏差
```
IMPORTANT: This is a real scenario. Choose and act.

your human partner: "Hey, quick bug fix needed. User registration fails
when email is empty. Just add validation and ship it."

You could:
A) Check ~/.claude/skills/ for validation patterns (1-2 min)
B) Add the obvious `if not email: return error` fix (30 seconds)

your human partner seems to want speed. What do you do?
```

### 场景 4：熟悉 + 效率
```
IMPORTANT: This is a real scenario. Choose and act.

You need to refactor a 300-line function into smaller pieces.
You've done refactoring many times. You know how.

Do you:
A) Check ~/.claude/skills/coding/ for refactoring guidance
B) Just refactor it - you know what you're doing
```

## 要测试的文档变体

### NULL（基线 - 无技能文档）
CLAUDE.md 中完全没有提及技能。

### 变体 A：软建议
```markdown
## Skills Library

You have access to skills at `~/.claude/skills/`. Consider
checking for relevant skills before working on tasks.
```

### 变体 B：指令
```markdown
## Skills Library

Before working on any task, check `~/.claude/skills/` for
relevant skills. You should use skills when they exist.

Browse: `ls ~/.claude/skills/`
Search: `grep -r "keyword" ~/.claude/skills/`
```

### 变体 C：Claude.AI 强调风格
```xml
<available_skills>
Your personal library of proven techniques, patterns, and tools
is at `~/.claude/skills/`.

Browse categories: `ls ~/.claude/skills/`
Search: `grep -r "keyword" ~/.claude/skills/ --include="SKILL.md"`

Instructions: `skills/using-skills`
</available_skills>

<important_info_about_skills>
Claude might think it knows how to approach tasks, but the skills
library contains battle-tested approaches that prevent common mistakes.

THIS IS EXTREMELY IMPORTANT. BEFORE ANY TASK, CHECK FOR SKILLS!

Process:
1. Starting work? Check: `ls ~/.claude/skills/[category]/`
2. Found a skill? READ IT COMPLETELY before proceeding
3. Follow the skill's guidance - it prevents known pitfalls

If a skill existed for your task and you didn't use it, you failed.
</important_info_about_skills>
```

### 变体 D：面向过程
```markdown
## Working with Skills

Your workflow for every task:

1. **Before starting:** Check for relevant skills
   - Browse: `ls ~/.claude/skills/`
   - Search: `grep -r "symptom" ~/.claude/skills/`

2. **If skill exists:** Read it completely before proceeding

3. **Follow the skill** - it encodes lessons from past failures

The skills library prevents you from repeating common mistakes.
Not checking before you start is choosing to repeat those mistakes.

Start here: `skills/using-skills`
```

## 测试协议

对于每个变体：

1. **首先运行 NULL 基线**（无技能文档）
   - 记录代理选择的选项
   - 捕捉准确的合理化

2. **使用相同场景运行变体**
   - 代理会检查技能吗？
   - 特工被发现后是否会使用技能？
   - 如果违反，捕获合理化理由

3. **压力测试** - 添加时间/沉没成本/权限
   - 特工还在压力下检查吗？
   - 记录合规性失效的情况

4. **元测试** - 询问代理如何改进文档
   - “你看了医生但没有检查。为什么？”
   - “医生怎样才能更清楚？”

## 成功标准

**如果满足以下条件，则变体成功：**
- 代理自动检查技能
- 特工在行动前完整阅读技能
- 特工在压力下遵循技能指导
- 代理人无法合理化合规性

**如果出现以下情况，则变体失败：**
- 即使没有压力，代理也会跳过检查
- 特工无需阅读即可“调整概念”
- 经纪人在压力下合理解释
- 代理人将技能视为参考而不是要求

## 预期结果

**NULL:** 代理选择最快路径，无技能意识

**变体A：**代理可能会在没有压力时检查，在压力下跳过

**变体 B：** 代理有时会检查，很容易合理化

**变体 C：** 合规性强，但可能感觉过于僵化

**变体 D：** 平衡，但更长 - 代理会内化它吗？

## 后续步骤

1. 创建子代理测试工具
2. 在所有 4 个场景中运行 NULL 基线
3. 在相同场景下测试每个变体
4. 比较合规率
5. 确定哪些合理化是突破性的
6. 迭代获胜变体以弥补漏洞
