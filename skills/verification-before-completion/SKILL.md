---
name: verification-before-completion
description: 在提交或创建 PR 之前，即将声明工作已完成、修复或通过时使用 - 需要在做出任何成功声明之前运行验证命令并确认输出；证据总是先于断言
---

# 完成前验证

## 概述

在没有验证的情况下声称工作已经完成是不诚实的，而不是效率。

**核心原则：** 证据先于主张，始终如此。

**违反本规则的字面意义即违反本规则的精神。**

## 铁律

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

如果您没有运行此消息中的验证命令，则不能声称它通过了。

## 门函数

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## 常见故障

|索赔 |需要|还不够|
|-------|----------|----------------|
|测试通过 |测试命令输出：0 次失败 |上一次运行，“应该通过” |
|棉绒清洁 | Linter 输出：0 个错误 |部分检查、推断 |
|构建成功|构建命令：exit 0 | Linter 通过，日志看起来不错 |
|错误已修复 |测试原始症状：通过 |代码已更改，假设已修复 |
|回归测试有效 |红绿循环验证 |测试一次通过 |
|代理完成 | VCS diff 显示更改 |代理报告“成功”|
|满足要求 |逐行检查表 |测试通过 |

## 危险信号 - 停止

- 使用“应该”、“可能”、“似乎”
- 在验证前表达满意（“太棒了！”、“完美！”、“完成！”等）
- 未经验证即将提交/推送/PR
- 信任代理成功报告
- 依赖部分验证
- 想着“就这一次”
- 累了并且想要结束工作
- **任何暗示成功而无需运行验证的措辞**

## 合理化预防

|对不起|现实|
|--------|---------|
| “现在应该工作了” |运行验证 |
| “我有信心” |信心≠证据|
| “就这一次”|无一例外 |
| “Linter 通过” | Linter ≠ 编译器 |
| 《代理说成功》|独立验证 |
| “我累了”|疲惫≠借口|
| “部分检查就足够了” |部分证明不了什么|
| “不同的词语，因此规则不适用” |精神重于文字|

## 关键模式

**测试：**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**回归测试（TDD 红-绿）：**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**建造：**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**要求：**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**代理委托：**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

## 为什么这很重要

24次失败记忆：
- 你的人类伴侣说“我不相信你” - 信任被打破
- 发布了未定义的函数 - 会崩溃
- 缺少已交付的需求 - 功能不完整
- 时间浪费在错误完成→重定向→返工上
- 违规：“诚实是核心价值观。如果你撒谎，你就会被取代。”

## 何时申请

**总是在：**之前
- 成功/完成声明的任何变化
- 任何满意的表达
- 关于工作状态的任何积极的陈述
- 承诺、公关创建、任务完成
- 转到下一个任务
- 委托给代理人

**规则适用于：**
- 准确的短语
- 释义和同义词
- 成功的意义
- 任何表明完成/正确性的通信

## 底线

**验证没有捷径。**

运行命令。读取输出。然后领取结果。

这是没有商量余地的。
