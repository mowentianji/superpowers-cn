# 测试反模式

**在以下情况下加载此参考：** 编写或更改测试、添加模拟或试图将仅测试方法添加到生产代码中。

## 概述

测试必须验证真实行为，而不是模拟行为。模拟是一种隔离手段，而不是被测试的东西。

**核心原则：** 测试代码做什么，而不是模拟做什么。

**遵循严格的 TDD 可以防止这些反模式。**

## 铁律

```
1. NEVER test mock behavior
2. NEVER add test-only methods to production classes
3. NEVER mock without understanding dependencies
```

## 反模式 1：测试模拟行为

**违规行为：**
```typescript
// ❌ BAD: Testing that the mock exists
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```

**为什么这是错误的：**
- 您正在验证模拟是否有效，而不是组件是否有效
- 当模拟存在时测试通过，当模拟不存在时测试失败
- 不告诉你任何关于真实行为的信息

**你的人类伙伴的更正：**“我们正在测试模拟的行为吗？”

**修复：**
```typescript
// ✅ GOOD: Test real component or don't mock it
test('renders sidebar', () => {
  render(<Page />);  // Don't mock sidebar
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});

// OR if sidebar must be mocked for isolation:
// Don't assert on the mock - test Page's behavior with sidebar present
```

### 门函数

```
BEFORE asserting on any mock element:
  Ask: "Am I testing real component behavior or just mock existence?"

  IF testing mock existence:
    STOP - Delete the assertion or unmock the component

  Test real behavior instead
```

## 反模式 2：生产中的仅测试方法

**违规行为：**
```typescript
// ❌ BAD: destroy() only used in tests
class Session {
  async destroy() {  // Looks like production API!
    await this._workspaceManager?.destroyWorkspace(this.id);
    // ... cleanup
  }
}

// In tests
afterEach(() => session.destroy());
```

**为什么这是错误的：**
- 生产类被仅测试代码污染
- 如果在生产中意外调用会很危险
- 违反了 YAGNI 和关注点分离
- 混淆了对象生命周期和实体生命周期

**修复：**
```typescript
// ✅ GOOD: Test utilities handle test cleanup
// Session has no destroy() - it's stateless in production

// In test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}

// In tests
afterEach(() => cleanupSession(session));
```

### 门函数

```
BEFORE adding any method to production class:
  Ask: "Is this only used by tests?"

  IF yes:
    STOP - Don't add it
    Put it in test utilities instead

  Ask: "Does this class own this resource's lifecycle?"

  IF no:
    STOP - Wrong class for this method
```

## 反模式 3：在不理解的情况下进行嘲笑

**违规行为：**
```typescript
// ❌ BAD: Mock breaks test logic
test('detects duplicate server', () => {
  // Mock prevents config write that test depends on!
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));

  await addServer(config);
  await addServer(config);  // Should throw - but won't!
});
```

**为什么这是错误的：**
- 模拟方法有依赖于副作用测试（编写配置）
- 过度嘲笑“安全”破坏了实际行为
- 测试因错误原因通过或神秘失败

**修复：**
```typescript
// ✅ GOOD: Mock at correct level
test('detects duplicate server', () => {
  // Mock the slow part, preserve behavior test needs
  vi.mock('MCPServerManager'); // Just mock slow server startup

  await addServer(config);  // Config written
  await addServer(config);  // Duplicate detected ✓
});
```

### 门函数

```
BEFORE mocking any method:
  STOP - Don't mock yet

  1. Ask: "What side effects does the real method have?"
  2. Ask: "Does this test depend on any of those side effects?"
  3. Ask: "Do I fully understand what this test needs?"

  IF depends on side effects:
    Mock at lower level (the actual slow/external operation)
    OR use test doubles that preserve necessary behavior
    NOT the high-level method the test depends on

  IF unsure what test depends on:
    Run test with real implementation FIRST
    Observe what actually needs to happen
    THEN add minimal mocking at the right level

  Red flags:
    - "I'll mock this to be safe"
    - "This might be slow, better mock it"
    - Mocking without understanding the dependency chain
```

## 反模式 4：不完整的模拟

**违规行为：**
```typescript
// ❌ BAD: Partial mock - only fields you think you need
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // Missing: metadata that downstream code uses
};

// Later: breaks when code accesses response.metadata.requestId
```

**为什么这是错误的：**
- **部分模拟隐藏了结构假设** - 您只模拟了您知道的字段
- **下游代码可能取决于您未包含的字段** - 无提示失败
- **测试通过，但集成失败** - 模拟不完整，真实 API 完整
- **虚假信心** - 测试无法证明真实行为

**铁律：** 模拟现实中存在的完整数据结构，而不仅仅是您直接测试使用的字段。

**修复：**
```typescript
// ✅ GOOD: Mirror real API completeness
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
  // All fields real API returns
};
```

### 门函数

```
BEFORE creating mock responses:
  Check: "What fields does the real API response contain?"

  Actions:
    1. Examine actual API response from docs/examples
    2. Include ALL fields system might consume downstream
    3. Verify mock matches real response schema completely

  Critical:
    If you're creating a mock, you must understand the ENTIRE structure
    Partial mocks fail silently when code depends on omitted fields

  If uncertain: Include all documented fields
```

## 反模式 5：事后才考虑集成测试

**违规行为：**
```
✅ Implementation complete
❌ No tests written
"Ready for testing"
```

**为什么这是错误的：**
- 测试是实施的一部分，而不是可选的后续行动
- TDD 会发现这个
- 未经测试不能声称完整

**修复：**
```
TDD cycle:
1. Write failing test
2. Implement to pass
3. Refactor
4. THEN claim complete
```

## 当模拟变得过于复杂时

**警告标志：**
- 模拟设置比测试逻辑长
- 模拟一切以使测试通过
- 模拟缺少真实组件所具有的方法
- 当模拟更改时测试中断

**你的人类伙伴的问题：**“我们需要在这里使用模拟吗？”

**考虑：** 与真实组件的集成测试通常比复杂的模拟更简单

## TDD 可以防止这些反模式

**为什么 TDD 有帮助：**
1. **首先编写测试** → 迫使您思考您实际测试的内容
2. **观察它失败** → 确认测试测试真实行为，而不是模拟
3. **最小实现** → 没有纯测试方法
4. **真正的依赖关系** → 在模拟之前您会看到测试实际需要什么

**如果您正在测试模拟行为，则违反了 TDD** - 您添加了模拟，而没有先观察真实代码的测试失败。

## 快速参考

|反模式 |修复|
|--------------|-----|
|对模拟元素进行断言 |测试真实组件或解锁它 |
|生产中的仅测试方法 |移至测试实用程序 |
|没有理解的嘲笑|首先了解依赖关系，最小化模拟 |
|不完整的模拟 |完全镜像真实API |
|事后才进行测试| TDD - 首先测试|
|过于复杂的模拟 |考虑集成测试 |

## 危险信号

- `*-mock` 测试 ID 的断言检查
- 仅在测试文件中调用的方法
- 模拟设置>测试的50%
- 删除模拟时测试失败
- 无法解释为什么需要模拟
- 嘲笑“只是为了安全”

## 底线

**模拟是隔离的工具，而不是测试的东西。**

如果 TDD 显示您正在测试模拟行为，那么您就错了。

修复：测试真实行为或质疑你为什么要嘲笑。
