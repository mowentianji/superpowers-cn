# 代码质量审查子代理模板

派发代码质量审查子代理时使用此模板。

**目的：** 验证实现方式是否足够优秀：代码干净、测试有效、结构可维护。

**只有在规格符合性审查通过后，才允许发起这一审查。**

```text
Task tool（superpowers:code-reviewer）:
  使用模板：requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [来自实现者汇报]
  PLAN_OR_REQUIREMENTS: [plan-file] 中的任务 N
  BASE_SHA: [任务开始前的提交]
  HEAD_SHA: [当前提交]
  DESCRIPTION: [任务摘要]
```

**代码审查子代理的返回内容应包括：** 优点、问题（严重 / 重要 / 次要）、总体评估。
