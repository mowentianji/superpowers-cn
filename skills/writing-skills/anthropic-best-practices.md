# 技能创作最佳实践

> 了解如何编写克劳德可以成功发现和使用的有效技能。

好的技能是简洁的、结构良好的，并且经过实际使用的测试。本指南提供实用的创作决策，帮助您编写 Claude 可以有效发现和使用的技能。

有关技能如何发挥作用的概念背景，请参阅[技能概述](/en/docs/agents-and-tools/agent-skills/overview)。

## 核心原则

### 简洁是关键

[上下文窗口](https://platform.claude.com/docs/en/build-with-claude/context-windows) 是公共物品。您的技能与克劳德需要知道的其他所有内容共享上下文窗口，包括：

* 系统提示
* 对话历史记录
* 其他技能的元数据
* 您的实际要求

并非您技能中的每个标记都会立即产生费用。启动时，仅预加载所有技能的元数据（名称和描述）。 Claude 仅在技能相关时才读取 SKILL.md，并且仅在需要时读取其他文件。然而，SKILL.md 中的简洁性仍然很重要：一旦 Claude 加载它，每个标记都会与对话历史记录和其他上下文竞争。

**默认假设**：克劳德已经很聪明了

只添加克劳德还没有的上下文。挑战每一条信息：

*“克劳德真的需要这个解释吗？”
*“我可以假设克劳德知道这一点吗？”
*“这一段是否证明其代币成本合理？”

**好例子：简洁**（大约 50 个标记）：

````markdown  theme={null}
## Extract PDF text

Use pdfplumber for text extraction:

```python
导入 pdfplumber

使用 pdfplumber.open("file.pdf") 作为 pdf：
    text = pdf.pages[0].extract_text()
```
````

**不好的例子：太冗长**（大约 150 个标记）：

```markdown  theme={null}
## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing, but we
recommend pdfplumber because it's easy to use and handles most cases well.
First, you'll need to install it using pip. Then you can use the code below...
```

简明版本假设 Claude 知道 PDF 是什么以及图书馆如何工作。

### 设置适当的自由度

将具体程度与任务的脆弱性和可变性相匹配。

**高自由度**（基于文本的说明）：

使用时：

* 多种方法均有效
* 决定取决于具体情况
* 启发式指导方法

示例：

```markdown  theme={null}
## Code review process

1. Analyze the code structure and organization
2. Check for potential bugs or edge cases
3. Suggest improvements for readability and maintainability
4. Verify adherence to project conventions
```

**中等自由度**（带有参数的伪代码或脚本）：

使用时：

* 存在首选模式
* 一些变化是可以接受的
* 配置影响行为

示例：

````markdown  theme={null}
## Generate report

Use this template and customize as needed:

```python
defgenerate_report（数据，格式=“markdown”，include_charts=True）：
    # Process data
    # Generate output in specified format
    # Optionally include visualizations
```
````

**低自由度**（特定脚本，很少或没有参数）：

使用时：

* 操作脆弱且容易出错
* 一致性至关重要
* 必须遵循特定的顺序

示例：

````markdown  theme={null}
## Database migration

Run exactly this script:

```bash
python 脚本/migrate.py --verify --backup
```

Do not modify the command or add additional flags.
````

**类比**：将克劳德想象成一个正在探索路径的机器人：

* **两边都是悬崖的窄桥**：只有一条安全的前进道路。提供具体的护栏和准确的指示（低自由度）。示例：必须按精确顺序运行的数据库迁移。
* **没有危险的开阔场地**：许多道路通向成功。给予大方向并信任克劳德找到最佳路线（高自由度）。示例：代码审查，其中上下文决定最佳方法。

### 使用您计划使用的所有模型进行测试

技能充当模型的补充，因此有效性取决于底层模型。使用您计划使用的所有模型来测试您的技能。

**按模型测试注意事项**：

* **克劳德俳句**（快速、经济）：该技能是否提供足够的指导？
* **Claude Sonnet**（平衡）：技能是否清晰且高效？
* **Claude Opus**（强有力的推理）：该技能是否避免过度解释？

对于 Opus 来说完美的方法可能对于 Haiku 来说需要更多细节。如果您计划在多个模型中使用您的技能，请瞄准适用于所有模型的说明。

## 技能结构

<Note>
  **YAML Frontmatter**：SKILL.md frontmatter 支持两个字段：

  * `name` - 人类可读的技能名称（最多 64 个字符）
  * `description` - 一行描述该技能的用途以及何时使用该技能（最多 1024 个字符）

  有关完整的技能结构详细信息，请参阅[技能概述](/en/docs/agents-and-tools/agent-skills/overview#skill-structure)。
</Note>

### 命名约定

使用一致的命名模式使技能更易于参考和讨论。我们建议使用**动名词形式**（动词 + -ing）作为技能名称，因为这清楚地描述了技能提供的活动或功能。

**良好的命名示例（动名词形式）**：

*“处理 PDF”
*“分析电子表格”
*“管理数据库”
*“测试代码”
*“编写文档”

**可接受的替代方案**：

* 名词短语：“PDF 处理”、“电子表格分析”
* 面向行动：“处理 PDF”、“分析电子表格”

**避免**：

* 模糊名称：“Helper”、“Utils”、“Tools”
* 过于通用：“文档”、“数据”、“文件”
* 你的技能集合中的模式不一致

一致的命名可以更轻松地：

* 文档和对话中的参考技巧
* 一目了然了解技能的作用
* 整理和搜索多种技能
* 维护专业、有凝聚力的技能库

### 撰写有效的描述

`description` 字段支持技能发现，并且应包括技能的用途和使用时间。

<Warning>
  **始终以第三人称写作**。描述被注入到系统提示中，不一致的观点会导致发现问题。

  * **好：**“处理 Excel 文件并生成报告”
  * **避免：** “我可以帮你处理Excel文件”
  * **避免：** “您可以使用它来处理 Excel 文件”
</Warning>

**具体并包括关键术语**。包括该技能的用途以及何时使用该技能的特定触发器/上下文。

每项技能都有一个描述字段。描述对于技能选择至关重要：Claude 使用它从潜在的 100 多个可用技能中选择正确的技能。您的描述必须提供足够的详细信息，以便 Claude 知道何时选择此技能，而 SKILL.md 的其余部分提供了实现细节。

有效例子：

**PDF处理技巧：**

```yaml  theme={null}
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**Excel分析技巧：**

```yaml  theme={null}
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Git 提交助手技能：**

```yaml  theme={null}
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

避免诸如此类的模糊描述：

```yaml  theme={null}
description: Helps with documents
```

```yaml  theme={null}
description: Processes data
```

```yaml  theme={null}
description: Does stuff with files
```

### 渐进式披露模式

SKILL.md 充当概述，根据需要为 Claude 提供详细材料，例如入门指南中的目录。有关渐进式披露如何发挥作用的说明，请参阅概述中的[技能如何发挥作用](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

**实用指导：**

* 将 SKILL.md 正文保持在 500 行以下，以获得最佳性能
* 当接近此限制时将内容拆分为单独的文件
* 使用以下模式有效地组织指令、代码和资源

#### 视觉概述：从简单到复杂

基本技能从包含元数据和说明的 SKILL.md 文件开始：

<img src="ZXQPH0001QXZ alt="Simple SKILL.md file showing YAML frontmatter and markdown body" data-og-width="2048" width="2048" data-og-height="1153" height="1153" data-path="images/agent-skills-simple-file.png" data-optimize="true" data-opv="3" srcset="ZXQPH0002QXZ 280w, ZXQPH0003QXZ 560w, ZXQPH0004QXZ 840w, ZXQPH0005QXZ 1100w, ZXQPH0006QXZ 1650w, ZXQPH0007QXZ 2500w" />

随着您的技能增长，您可以捆绑克劳德仅在需要时加载的其他内容：

<img src="ZXQPH0008QXZ alt="Bundling additional reference files like reference.md and forms.md." data-og-width="2048" width="2048" data-og-height="1327" height="1327" data-path="images/agent-skills-bundling-content.png" data-optimize="true" data-opv="3" srcset="ZXQPH0009QXZ 280w, ZXQPH0010QXZ 560w, ZXQPH0011QXZ 840w, ZXQPH0012QXZ 1100w, ZXQPH0013QXZ 1650w, ZXQPH0014QXZ 2500w" />

完整的 Skill 目录结构可能如下所示：

```
pdf/
├── SKILL.md              # Main instructions (loaded when triggered)
├── FORMS.md              # Form-filling guide (loaded as needed)
├── reference.md          # API reference (loaded as needed)
├── examples.md           # Usage examples (loaded as needed)
└── scripts/
    ├── analyze_form.py   # Utility script (executed, not loaded)
    ├── fill_form.py      # Form filling script
    └── validate.py       # Validation script
```

#### 模式 1：带有参考资料的高级指南

````markdown  theme={null}
---
name: PDF Processing
description: Extracts text and tables from PDF files, fills forms, and merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---

# PDF Processing

## Quick start

Extract text with pdfplumber:
```python
导入 pdfplumber
使用 pdfplumber.open("file.pdf") 作为 pdf：
    text = pdf.pages[0].extract_text()
```

## Advanced features

**Form filling**: See [FORMS.md](FORMS.md) for complete guide
**API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
**Examples**: See [EXAMPLES.md](EXAMPLES.md) for common patterns
````

Claude 仅在需要时加载 FORMS.md、REFERENCE.md 或 Examples.md。

#### 模式 2：特定领域的组织

对于具有多个领域的技能，请按领域组织内容，以避免加载不相关的上下文。当用户询问销售指标时，Claude 只需读取与销售相关的模式，而不是财务或营销数据。这使得令牌使用量保持在较低水平并集中于上下文。

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

````markdown SKILL.md theme={null}
# BigQuery Data Analysis

## Available datasets

**Finance**: Revenue, ARR, billing → See [reference/finance.md](reference/finance.md)
**Sales**: Opportunities, pipeline, accounts → See [reference/sales.md](reference/sales.md)
**Product**: API usage, features, adoption → See [reference/product.md](reference/product.md)
**Marketing**: Campaigns, attribution, email → See [reference/marketing.md](reference/marketing.md)

## Quick search

Find specific metrics using grep:

```bash
grep -i“收入”参考/finance.md
grep -i“管道”参考/sales.md
grep -i "api 使用" 参考/product.md
```
````

#### 模式 3：条件细节

显示基本内容，高级内容链接：

```markdown  theme={null}
# DOCX Processing

## Creating documents

Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents

For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

仅当用户需要这些功能时，Claude 才会读取 REDLINING.md 或 OOXML.md。

### 避免深层嵌套引用

当从其他引用的文件引用文件时，克劳德可能会部分读取文件。当遇到嵌套引用时，Claude 可能会使用 `head -100` 等命令来预览内容，而不是读取整个文件，从而导致信息不完整。

**将 SKILL.md 中的引用保持深一层**。所有参考文件应直接从 SKILL.md 链接，以确保 Claude 在需要时读取完整的文件。

**坏例子：太深了**：

```markdown  theme={null}
# SKILL.md
See [advanced.md](advanced.md)...

# advanced.md
See [details.md](details.md)...

# details.md
Here's the actual information...
```

**好例子：一层深**：

```markdown  theme={null}
# SKILL.md

**Basic usage**: [instructions in SKILL.md]
**Advanced features**: See [advanced.md](advanced.md)
**API reference**: See [reference.md](reference.md)
**Examples**: See [examples.md](examples.md)
```

### 使用目录构建较长的参考文件

对于超过 100 行的参考文件，请在顶部包含目录。这确保了克劳德即使在部分读取预览时也能看到完整的可用信息。

**例子**：

```markdown  theme={null}
# API Reference

## Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features (batch operations, webhooks)
- Error handling patterns
- Code examples

## Authentication and setup
...

## Core methods
...
```

然后，克劳德可以读取完整的文件或根据需要跳转到特定部分。

有关此基于文件系统的架构如何实现渐进式公开的详细信息，请参阅下面“高级”部分中的“运行时环境](#runtime-environment)”部分。

## 工作流程和反馈循环

### 使用工作流程来完成复杂的任务

将复杂的操作分解为清晰、连续的步骤。对于特别复杂的工作流程，请提供一个清单，克劳德可以将其复制到其响应中并在进展过程中进行核对。

**示例 1：研究综合工作流程**（适用于无代码的技能）：

````markdown  theme={null}
## Research synthesis workflow

Copy this checklist and track your progress:

```
研究进展：
- [ ] 步骤1：阅读所有源文档
- [ ] 第 2 步：确定关键主题
- [ ] 步骤 3：交叉引用权利要求
- [ ] 步骤 4：创建结构化摘要
- [ ] 步骤 5：验证引文
```

**Step 1: Read all source documents**

Review each document in the `sources/` directory. Note the main arguments and supporting evidence.

**Step 2: Identify key themes**

Look for patterns across sources. What themes appear repeatedly? Where do sources agree or disagree?

**Step 3: Cross-reference claims**

For each major claim, verify it appears in the source material. Note which source supports each point.

**Step 4: Create structured summary**

Organize findings by theme. Include:
- Main claim
- Supporting evidence from sources
- Conflicting viewpoints (if any)

**Step 5: Verify citations**

Check that every claim references the correct source document. If citations are incomplete, return to Step 3.
````

此示例展示了工作流如何应用于不需要代码的分析任务。检查表模式适用于任何复杂的多步骤流程。

**示例 2：PDF 表单填写工作流程**（针对代码技能）：

````markdown  theme={null}
## PDF form filling workflow

Copy this checklist and check off items as you complete them:

```
任务进展：
- [ ] 步骤1：分析表单（运行analyze_form.py）
- [ ] 步骤 2：创建字段映射（编辑 fields.json）
- [ ] 步骤 3：验证映射（运行 validate_fields.py）
- [ ] 第 4 步：填写表格（运行 fill_form.py）
- [ ] 步骤 5：验证输出（运行 verify_output.py）
```

**Step 1: Analyze the form**

Run: `python scripts/analyze_form.py input.pdf`

This extracts form fields and their locations, saving to `fields.json`.

**Step 2: Create field mapping**

Edit `fields.json` to add values for each field.

**Step 3: Validate mapping**

Run: `python scripts/validate_fields.py fields.json`

Fix any validation errors before continuing.

**Step 4: Fill the form**

Run: `python scripts/fill_form.py input.pdf fields.json output.pdf`

**Step 5: Verify output**

Run: `python scripts/verify_output.py output.pdf`

If verification fails, return to Step 2.
````

清晰的步骤可防止克劳德跳过关键验证。该清单可帮助 Claude 和您跟踪多步骤工作流程的进度。

### 实施反馈循环

**常见模式**：运行验证器→修复错误→重复

这种模式极大地提高了输出质量。

**示例 1：风格指南合规性**（对于没有代码的技能）：

```markdown  theme={null}
## Content review process

1. Draft your content following the guidelines in STYLE_GUIDE.md
2. Review against the checklist:
   - Check terminology consistency
   - Verify examples follow the standard format
   - Confirm all required sections are present
3. If issues found:
   - Note each issue with specific section reference
   - Revise the content
   - Review the checklist again
4. Only proceed when all requirements are met
5. Finalize and save the document
```

这显示了使用参考文档而不是脚本的验证循环模式。 “验证器”是STYLE\_GUIDE.md，Claude通过阅读和比较来执行检查。

**示例 2：文档编辑过程**（针对代码技能）：

```markdown  theme={null}
## Document editing process

1. Make your edits to `word/document.xml`
2. **Validate immediately**: `python ooxml/scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Review the error message carefully
   - Fix the issues in the XML
   - Run validation again
4. **Only proceed when validation passes**
5. Rebuild: `python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. Test the output document
```

验证循环尽早发现错误。

## 内容指南

### 避免时间敏感的信息

不要包含会过时的信息：

**坏例子：时间敏感**（会出错）：

```markdown  theme={null}
If you're doing this before August 2025, use the old API.
After August 2025, use the new API.
```

**好例子**（使用“旧模式”部分）：

```markdown  theme={null}
## Current method

Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns

<details>
<summary>Legacy v1 API (deprecated 2025-08)</summary>

The v1 API used: `api.example.com/v1/messages`

This endpoint is no longer supported.
</details>
```

旧模式部分提供了历史背景，但又不会扰乱主要内容。

### 使用一致的术语

选择一个术语并在整个技能中使用它：

**好 - 一致**：

* 始终为“API端点”
* 总是“田野”
* 始终“提取”

**不好 - 不一致**：

* 混合“API端点”、“URL”、“API路由”、“路径”
* 混合“字段”、“框”、“元素”、“控件”
* 混合“提取”、“拉取”、“获取”、“检索”

一致性有助于克劳德理解并遵循指示。

## 常见模式

### 模板模式

提供输出格式的模板。将严格程度与您的需求相匹配。

**对于严格要求**（例如 API 响应或数据格式）：

````markdown  theme={null}
## Report structure

ALWAYS use this exact template structure:

```markdown
# [分析标题]

## 执行摘要
[主要发现的一段概述]

## 主要发现
- 找到 1 并提供支持数据
- 发现 2 并提供支持数据
- 发现 3 并提供支持数据

## 建议
1. 具体可行的建议
2. 具体可行的建议
```
````

**为了灵活指导**（当适应有用时）：

````markdown  theme={null}
## Report structure

Here is a sensible default format, but use your best judgment based on the analysis:

```markdown
# [分析标题]

## 执行摘要
[概述]

## 主要发现
[根据您的发现调整部分]

## 建议
【根据具体情况量身定制】
```

Adjust sections as needed for the specific analysis type.
````

### 示例模式

对于输出质量取决于查看示例的技能，请像常规提示一样提供输入/输出对：

````markdown  theme={null}
## Commit message format

Generate commit messages following these examples:

**Example 1:**
Input: Added user authentication with JWT tokens
Output:
```
feat(auth)：实现基于 JWT 的身份验证

添加登录端点和令牌验证中间件
```

**Example 2:**
Input: Fixed bug where dates displayed incorrectly in reports
Output:
```
修复（报告）：时区转换中的正确日期格式

在报告生成过程中一致使用 UTC 时间戳
```

**Example 3:**
Input: Updated dependencies and refactored error handling
Output:
```
杂务：更新依赖关系并重构错误处理

- 将 lodash 升级至 4.17.21
- 跨端点标准化错误响应格式
```

Follow this style: type(scope): brief description, then detailed explanation.
````

与单独的描述相比，示例可以帮助克劳德更清楚地理解所需的风格和详细程度。

### 条件工作流程模式

引导克劳德完成决策点：

```markdown  theme={null}
## Document modification workflow

1. Determine the modification type:

   **Creating new content?** → Follow "Creation workflow" below
   **Editing existing content?** → Follow "Editing workflow" below

2. Creation workflow:
   - Use docx-js library
   - Build document from scratch
   - Export to .docx format

3. Editing workflow:
   - Unpack existing document
   - Modify XML directly
   - Validate after each change
   - Repack when complete
```

<Tip>
  如果工作流程变得庞大或复杂且包含许多步骤，请考虑将它们放入单独的文件中，并告诉 Claude 根据手头的任务读取适当的文件。
</Tip>

## 评估和迭代

### 首先建立评估

**在编写大量文档之前创建评估。**这可确保您的技能解决实际问题，而不是记录想象的问题。

**评估驱动的开发：**

1. **找出差距**：让克劳德执行没有技能的代表性任务。记录特定的失败或缺失的上下文
2. **创建评估**：构建三个场景来测试这些差距
3. **建立基线**：在没有技能的情况下衡量克劳德的表现
4. **编写最少的说明**：创建足够的内容来弥补差距并通过评估
5. **迭代**：执行评估、与基线进行比较并完善

这种方法可确保您解决实际问题，而不是预测可能永远不会实现的需求。

**评估结构**：

```json  theme={null}
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF file and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully reads the PDF file using an appropriate PDF processing library or command-line tool",
    "Extracts text content from all pages in the document without missing any pages",
    "Saves the extracted text to a file named output.txt in a clear, readable format"
  ]
}
```

<Note>
  此示例演示了使用简单测试规则的数据驱动评估。我们目前不提供运行这些评估的内置方法。用户可以创建自己的评价体系。评估是衡量技能有效性的事实来源。
</Note>

### 与 Claude 一起迭代开发技能

最有效的技能开发过程涉及克劳德本身。与克劳德的一个实例（“克劳德 A”）合作创建一项将由其他实例（“克劳德 B”）使用的技能。 Claude A 帮助您设计和完善指令，而 Claude B 在实际任务中测试它们。这是有效的，因为 Claude 模型了解如何编写有效的代理指令以及代理需要什么信息。

**创建新技能：**

1. **无需技能即可完成任务**：使用正常提示与 Claude A 一起解决问题。在工作时，您自然会提供背景信息、解释偏好并分享程序知识。注意您反复提供的信息。

2. **确定可重用模式**：完成任务后，确定您提供的上下文对未来类似的任务有用。

   **示例**：如果您进行了 BigQuery 分析，您可能已经提供了表名称、字段定义、过滤规则（例如“始终排除测试帐户”）和常见查询模式。

3. **要求 Claude A 创建一项技能**：“创建一项技能来捕获我们刚刚使用的 BigQuery 分析模式。包括表架构、命名约定以及有关过滤测试帐户的规则。”

   <Tip>
     Claude models understand the Skill format and structure natively. You don't need special system prompts or a "writing skills" skill to get Claude to help create Skills. Simply ask Claude to create a Skill and it will generate properly structured SKILL.md content with appropriate frontmatter and body content.
   </Tip>

4. **审查简洁性**：检查Claude A没有添加不必要的解释。问：“删除关于胜率意味着什么的解释——克劳德已经知道了。”

5. **改进信息架构**：要求Claude A更有效地组织内容。例如：“对其进行组织，以便表架构位于单独的参考文件中。稍后我们可能会添加更多表。”

6. **对类似任务进行测试**：在相关用例上使用 Claude B 的技能（已加载技能的新实例）。观察 Claude B 是否找到正确的信息、正确应用规则并成功处理任务。

7. **基于观察进行迭代**：如果 Claude B 遇到困难或错过了某些内容，请返回给 Claude A 并提供具体信息：“当 Claude 使用此技能时，它忘记按 Q4 的日期进行过滤。我们是否应该添加有关日期过滤模式的部分？”

**迭代现有技能：**

当提高技能时，同样的层次模式继续存在。您可以交替使用：

* **与Claude A**（帮助完善技能的专家）一起工作
* **与 Claude B 一起测试**（使用技能执行实际工作的代理）
* **观察克劳德 B 的行为** 并将见解带回给克劳德 A

1. **在真实工作流程中使用技能**：给 Claude B（已加载技能）实际任务，而不是测试场景

2. **观察 Claude B 的行为**：注意它在哪里挣扎、成功或做出意想不到的选择

   **示例观察**：“当我向 Claude B 索要区域销售报告时，它编写了查询，但忘记过滤掉测试帐户，即使技能提到了此规则。”

3. **返回 Claude A 寻求改进**：分享当前的 SKILL.md 并描述您观察到的内容。问：“我在要求提供区域报告时注意到 Claude B 忘记过滤测试帐户。技能中提到了过滤，但可能还不够突出？”

4. **回顾 Claude A 的建议**：Claude A 可能建议重新组织以使规则更加突出，使用“必须过滤”而不是“始终过滤”等更强的语言，或者重组工作流程部分。

5. **应用并测试更改**：使用 Claude A 的改进更新技能，然后根据类似的请求再次使用 Claude B 进行测试

6. **根据使用情况重复**：当遇到新场景时，继续此观察-优化-测试循环。每次迭代都会根据真实的代理行为而不是假设来提高技能。

**收集团队反馈：**

1.与队友分享技能并观察其使用情况
2. 问：技能是否按预期激活？指示是否清楚？缺少什么？
3. 纳入反馈以解决您自己的使用模式中的盲点

**为什么这种方法有效**：Claude A 了解代理需求，您提供领域专业知识，Claude B 通过实际使用揭示差距，迭代细化基于观察到的行为而不是假设来提高技能。

### 观察克劳德如何驾驭技能

当你迭代技能时，请注意克劳德在实践中如何实际使用它们。注意：

* **意外的探索路径**：克劳德是否以您没有预料到的顺序读取文件？这可能表明您的结构并不像您想象的那么直观
* **错过连接**：克劳德是否未能遵循对重要文件的引用？您的链接可能需要更明确或更突出
* **过度依赖某些部分**：如果 Claude 重复读取同一个文件，请考虑该内容是否应该放在主 SKILL.md 中
* **忽略的内容**：如果 Claude 从不访问捆绑文件，则它可能是不必要的或在主要说明中信号不佳

基于这些观察而不是假设进行迭代。技能元数据中的“名称”和“描述”尤其重要。克劳德在决定是否触发技能来响应当前任务时使用这些。确保它们清楚地描述了该技能的用途以及何时使用该技能。

## 要避免的反模式

### 避免 Windows 风格的路径

始终在文件路径中使用正斜杠，即使在 Windows 上也是如此：

* ✓ **好**：`scripts/helper.py`、`reference/guide.md`
* ✗ **避免**：`scripts\helper.py`、`reference\guide.md`

Unix 样式路径适用于所有平台，而 Windows 样式路径会在 Unix 系统上导致错误。

### 避免提供太多选择

除非必要，否则不要提出多种方法：

````markdown  theme={null}
**Bad example: Too many choices** (confusing):
"You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or..."

**Good example: Provide a default** (with escape hatch):
"Use pdfplumber for text extraction:
```python
导入 pdfplumber
```

For scanned PDFs requiring OCR, use pdf2image with pytesseract instead."
````

## 高级：可执行代码的技能

以下部分重点介绍包含可执行脚本的技能。如果您的技能仅使用 Markdown 指令，请跳至[有效技能清单](#checklist-for-effective-skills)。

### 解决，不要弃踢

在为技能编写脚本时，请处理错误情况，而不是向 Claude 下注。

**好例子：明确处理错误**：

```python  theme={null}
def process_file(path):
    """Process a file, creating it if it doesn't exist."""
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # Create file with default content instead of failing
        print(f"File {path} not found, creating default")
        with open(path, 'w') as f:
            f.write('')
        return ''
    except PermissionError:
        # Provide alternative instead of failing
        print(f"Cannot access {path}, using default")
        return ''
```

**坏例子：对克劳德踢球**：

```python  theme={null}
def process_file(path):
    # Just fail and let Claude figure it out
    return open(path).read()
```

配置参数也应该合理化并记录在案，以避免“巫术常数”（奥斯特豪特定律）。如果你不知道正确的值，克劳德将如何确定它？

**很好的例子：自我记录**：

```python  theme={null}
# HTTP requests typically complete within 30 seconds
# Longer timeout accounts for slow connections
REQUEST_TIMEOUT = 30

# Three retries balances reliability vs speed
# Most intermittent failures resolve by the second retry
MAX_RETRIES = 3
```

**坏例子：幻数**：

```python  theme={null}
TIMEOUT = 47  # Why 47?
RETRIES = 5   # Why 5?
```

### 提供实用脚本

即使克劳德可以编写脚本，预制脚本也具有以下优点：

**实用程序脚本的优点**：

* 比生成的代码更可靠
* 保存令牌（不需要在上下文中包含代码）
* 节省时间（无需生成代码）
* 确保不同用途的一致性

<img src="ZXQPH0003QXZ alt="Bundling executable scripts alongside instruction files" data-og-width="2048" width="2048" data-og-height="1154" height="1154" data-path="images/agent-skills-executable-scripts.png" data-optimize="true" data-opv="3" srcset="ZXQPH0004QXZ 280w, ZXQPH0005QXZ 560w, ZXQPH0006QXZ 840w, ZXQPH0007QXZ 1100w, ZXQPH0008QXZ 1650w, ZXQPH0009QXZ 2500w" />

上图显示了可执行脚本如何与指令文件一起工作。指令文件 (forms.md) 引用该脚本，Claude 可以执行它而无需将其内容加载到上下文中。

**重要区别**：在您的说明中明确克劳德是否应该：

* **执行脚本**（最常见）：“运行 `analyze_form.py` 提取字段”
* **将其作为参考**（对于复杂逻辑）：“有关字段提取算法，请参阅 `analyze_form.py`”

对于大多数实用程序脚本，首选执行，因为它更可靠、更高效。有关脚本执行工作原理的详细信息，请参阅下面的[运行时环境](#runtime-environment)]部分。

**例子**：

````markdown  theme={null}
## Utility scripts

**analyze_form.py**: Extract all form fields from PDF

```bash
python 脚本/analyze_form.py input.pdf > fields.json
```

Output format:
```json
{
  “字段名称”：{“类型”：“文本”，“x”：100，“y”：200}，
  “签名”：{“类型”：“sig”，“x”：150，“y”：500}
}
```

**validate_boxes.py**: Check for overlapping bounding boxes

```bash
python 脚本/validate_boxes.py fields.json
# 返回：“OK”或列出冲突
```

**fill_form.py**: Apply field values to PDF

```bash
python 脚本/fill_form.py input.pdf fields.json 输出.pdf
```
````

### 使用视觉分析

当输入可以呈现为图像时，让 Claude 分析它们：

````markdown  theme={null}
## Form layout analysis

1. Convert PDF to images:
   ```bash
   python脚本/pdf_to_images.py form.pdf
   ```

2. Analyze each page image to identify form fields
3. Claude can see field locations and types visually
````

<Note>
  在此示例中，您需要编写 `pdf_to_images.py` 脚本。
</Note>

克劳德的视觉能力有助于理解布局和结构。

### 创建可验证的中间输出

当克劳德执行复杂的、开放式的任务时，它可能会犯错误。 “计划-验证-执行”模式通过让克劳德首先以结构化格式创建计划，然后在执行之前使用脚本验证该计划，从而尽早捕获错误。

**示例**：想象一下要求 Claude 根据电子表格更新 PDF 中的 50 个表单字段。如果没有验证，Claude 可能会引用不存在的字段、创建冲突的值、错过必填字段或错误地应用更新。

**解决方案**：使用上面显示的工作流程模式（PDF 表单填写），但添加一个中间 `changes.json` 文件，该文件在应用更改之前进行验证。工作流程变为：分析 → **创建计划文件** → **验证计划** → 执行 → 验证。

**为什么这种模式有效：**

* **尽早发现错误**：验证在应用更改之前发现问题
* **机器可验证**：脚本提供客观验证
* **可逆规划**：克劳德可以在不触及原始计划的情况下迭代计划
* **清晰调试**：错误消息指向特定问题

**何时使用**：批量操作、破坏性更改、复杂的验证规则、高风险操作。

**实施提示**：使用特定的错误消息（例如“未找到字段‘signature\_date’。可用字段：customer\_name、order\_total、signature\_date\_signed”）使验证脚本变得详细，以帮助 Claude 解决问题。

### 包依赖项

技能在代码执行环境中运行，具有特定于平台的限制：

* **claude.ai**：可以从 npm 和 PyPI 安装软件包并从 GitHub 存储库中提取
* **Anthropic API**：没有网络访问权限，也没有运行时包安装

在 SKILL.md 中列出所需的软件包，并验证它们在[代码执行工具文档](/en/docs/agents-and-tools/tool-use/code-execution-tool)中是否可用。

### 运行时环境

技能在具有文件系统访问、bash 命令和代码执行功能的代码执行环境中运行。该架构的概念解释请参见概述中的【技能架构](/en/docs/agents-and-tools/agent-skills/overview#the-skills-architecture)】。

**这如何影响您的创作：**

**克劳德如何获得技能：**

1. **元数据预加载**：启动时，所有技能的 YAML frontmatter 中的名称和描述都会加载到系统提示符中
2. **文件按需读取**：Claude 使用 bash Read 工具在需要时从文件系统访问 SKILL.md 和其他文件
3. **高效执行脚本**：实用程序脚本可以通过 bash 执行，而无需将其完整内容加载到上下文中。只有脚本的输出消耗令牌
4. **大文件没有上下文惩罚**：参考文件、数据或文档在实际读取之前不会消耗上下文标记

* **文件路径很重要**：Claude 像文件系统一样导航您的技能目录。使用正斜杠 (`reference/guide.md`)，而不是反斜杠
* **以描述性方式命名文件**：使用指示内容的名称：`form_validation_rules.md`，而不是 `doc2.md`
* **组织发现**：按域或功能构建目录
  * 好：`reference/finance.md`、`reference/sales.md`
  * 不良：`docs/file1.md`、`docs/file2.md`
* **捆绑全面的资源**：包括完整的API文档、广泛的示例、大型数据集；在访问之前没有上下文惩罚
* **首选用于确定性操作的脚本**：编写 `validate_form.py` 而不是要求 Claude 生成验证代码
* **明确执行意图**：
  *“运行`analyze_form.py`提取字段”（执行）
  *“提取算法参见`analyze_form.py`”（作为参考阅读）
* **测试文件访问模式**：通过测试真实请求来验证 Claude 是否可以导航您的目录结构

**示例：**

```
bigquery-skill/
├── SKILL.md (overview, points to reference files)
└── reference/
    ├── finance.md (revenue metrics)
    ├── sales.md (pipeline data)
    └── product.md (usage analytics)
```

当用户询问收入时，Claude 会读取 SKILL.md，看到对 `reference/finance.md` 的引用，然后调用 bash 来读取该文件。 sales.md 和 Product.md 文件保留在文件系统上，在需要之前消耗零上下文令牌。这种基于文件系统的模型使得渐进式公开成为可能。克劳德可以导航并有选择地加载每个任务所需的内容。

有关技术架构的完整详细信息，请参阅技能概述中的[技能如何工作](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)]。

### MCP 工具参考

如果您的技能使用 MCP（模型上下文协议）工具，请始终使用完全限定的工具名称，以避免“找不到工具”错误。

**格式**：`ServerName:tool_name`

**示例**：

```markdown  theme={null}
Use the BigQuery:bigquery_schema tool to retrieve table schemas.
Use the GitHub:create_issue tool to create issues.
```

其中：

* `BigQuery` 和 `GitHub` 是 MCP 服务器名称
* `bigquery_schema` 和 `create_issue` 是这些服务器内的工具名称

如果没有服务器前缀，Claude 可能无法找到该工具，特别是当有多个 MCP 服务器可用时。

### 避免假设工具已安装

不要假设软件包可用：

````markdown  theme={null}
**Bad example: Assumes installation**:
"Use the pdf library to process the file."

**Good example: Explicit about dependencies**:
"Install required package: `pip install pypdf`

Then use it:
```python
从 pypdf 导入 PdfReader
reader = PdfReader("文件.pdf")
```"
````

## 技术说明

### YAML frontmatter 要求

SKILL.md frontmatter 仅包含 `name`（最多 64 个字符）和 `description`（最多 1024 个字符）字段。完整的结构细节请参见【技能​​概述](/en/docs/agents-and-tools/agent-skills/overview#skill-structure)】。

### 代币预算

将 SKILL.md 正文保持在 500 行以下，以获得最佳性能。如果您的内容超出此范围，请使用前面描述的渐进式披露模式将其拆分为单独的文件。架构详情请参见【技能​​概述](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)】。

## 有效技能清单

在分享技能之前，请验证：

### 核心品质

* [ ] 描述具体并包含关键术语
* [ ] 描述包括技能的用途和使用时间
* [ ] SKILL.md 正文低于 500 行
* [ ] 其他详细信息位于单独的文件中（如果需要）
* [ ] 没有时间敏感信息（或在“旧模式”部分）
* [ ] 始终使用一致的术语
* [ ] 例子是具体的，而不是抽象的
* [ ] 文件引用深一层
* [ ] 适当使用渐进式披露
* [ ] 工作流程步骤清晰

### 代码和脚本

* [ ] 脚本解决问题而不是向 Claude 下注
* [ ] 错误处理是明确且有帮助的
* [ ] 没有“巫毒常量”（所有值均合理）
* [ ] 说明中列出了所需的软件包并验证为可用
* [ ] 脚本有清晰的文档
* [ ] 没有 Windows 风格的路径（全部是正斜杠）
* [ ] 关键操作的验证/验证步骤
* [ ] 包含针对质量关键任务的反馈循环

### 测试

* [ ] 至少创建了三个评估
* [ ] 使用 Haiku、Sonnet 和 Opus 进行测试
* [ ] 经过真实使用场景测试
* [ ] 纳入团队反馈（如果适用）

## 后续步骤

<CardGroup cols={2}>
  <Card title="Get started with Agent Skills" icon="rocket" href="/en/docs/agents-and-tools/agent-skills/quickstart">
    Create your first Skill
  </Card>

  <Card title="Use Skills in Claude Code" icon="terminal" href="/en/docs/claude-code/skills">
    Create and manage Skills in Claude Code
  </Card>

  <Card title="Use Skills with the API" icon="code" href="/en/api/skills-guide">
    Upload and use Skills programmatically
  </Card>
</CardGroup>
