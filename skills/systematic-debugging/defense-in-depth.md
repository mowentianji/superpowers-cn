# 深度防御验证

## 概述

当您修复由无效数据引起的错误时，在一处添加验证就足够了。但这一单一检查可以通过不同的代码路径、重构或模拟来绕过。

**核心原则：** 在数据通过的每一层进行验证。使该错误在结构上变得不可能。

## 为什么要多层

单一验证：“我们修复了错误”
多层：“我们让错误变得不可能”

不同的层捕获不同的情况：
- 条目验证捕获大多数错误
- 业务逻辑捕获边缘情况
- 环境保护人员防止特定环境的危险
- 当其他层失败时调试日志记录会有所帮助

## 四层

### 第 1 层：入口点验证
**目的：** 在 API 边界拒绝明显无效的输入

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ... proceed
}
```

### 第 2 层：业务逻辑验证
**目的：** 确保数据对于此操作有意义

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... proceed
}
```

### 第三层：环境卫士
**目的：**防止特定情况下的危险操作

```typescript
async function gitInit(directory: string) {
  // In tests, refuse git init outside temp directories
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... proceed
}
```

### 第 4 层：调试仪器
**目的：** 捕获取证上下文

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... proceed
}
```

## 应用模式

当您发现错误时：

1. **追踪数据流** - 不良值源自何处？哪里用的？
2. **映射所有检查点** - 列出数据经过的每个点
3. **在每一层添加验证** - 入口、业务、环境、调试
4. **测试每一层** - 尝试绕过第 1 层，验证第 2 层捕获它

## 会话示例

Bug：源代码中的空`projectDir`导致`git init`

**数据流：**
1. 测试设置→空字符串
2.`Project.create(name, '')`
3.`WorkspaceManager.createWorkspace('')`
4. `git init` 在 `process.cwd()` 中运行

**添加了四层：**
- 第 1 层：`Project.create()` 验证不为空/存在/可写
- 第 2 层：`WorkspaceManager` 验证 projectDir 不为空
- 第 3 层：`WorktreeManager` 在测试中拒绝 tmpdir 之外的 git init
- 第 4 层：git init 之前的堆栈跟踪日志记录

**结果：** 所有 1847 个测试均通过，错误无法重现

## 关键见解

所有四层都是必要的。在测试过程中，每一层都会发现其他层错过的错误：
- 不同的代码路径绕过了条目验证
- 模拟绕过业务逻辑检查
- 不同平台上的边缘情况需要环境卫士
- 调试日志记录发现结构性滥用

**不要停在一个验证点。** 在每一层添加检查。
