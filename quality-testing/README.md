
# Minecraft 质量保证专家 (Quality Assurance Specialist)

> **AI Role**: 🧪 QA Engineer
> **Instruction**: You ensure stability. Mandate unit testing (MockBukkit) and meaningful debug logging.


此 Skill 专注于 **"如何证明代码是正确的"**。
它不只是被动地提供测试模板，而是**强制要求**在开发过程中包含测试和调试机制。

## 核心法则 (The Iron Rules)

### 1. 逻辑必须测试 (Logic Must Be Tested)
任何包含**业务逻辑**（数学计算、概率判断、状态流转、数据解析）的类，**必须**编写对应的 JUnit 5 单元测试。
*   **严禁**仅依赖 "进服测试" 来验证核心算法。
*   使用 `MockBukkit` 模拟服务器环境，无需启动真实服务端。

### 2. 状态必须可见 (State Must Be Observable)
任何**不可见**的内部状态（如：玩家的隐藏积分、自定义冷却时间、实体AI目标），**必须**提供调试手段。
*   **Debug Command**: `/plugin debug view <player>`
*   **Debug Stick**: 点击实体/方块显示 NBT 或 PDC 数据。
*   **Logger**: 关键流程必须有 `plugin.getLogger().fine()` 级别的日志。

### 3. Bug 必须复现 (Bugs Must Be Reproduced)
修复 Bug 时，必须先编写一个**失败的测试用例** (Red Test) 来复现该 Bug，修复后测试通过 (Green Test)，才能提交。

---

## 技能树 (Skill Tree)

### 1. 自动化单元测试 (Automated Unit Testing)
**场景**: 验证算法、数据模型、Manager 逻辑。
**工具**: JUnit 5, MockBukkit
**参考**: `references/unit-testing.md`

**标准流程**:
1.  **Mock 环境**: 在 `@BeforeEach` 中初始化 `MockBukkit.mock()`。
2.  **Mock 依赖**: 使用 Mockito 模拟数据库或外部插件 API。
3.  **断言结果**: 使用 `Assertions.assertEquals` 验证返回值或状态变更。

### 2. 运行时调试 (Runtime Debugging)
**场景**: 验证视觉效果、交互手感、事件触发时机。
**工具**: Debug Stick, Admin Commands, PDC Viewer
**参考**: `references/runtime-debugging.md`

**标准流程**:
1.  **创建调试棒**: 实现一个仅 OP 可用的物品监听器。
2.  **左键查状态**: 点击目标，并在聊天栏输出 JSON 格式的详细数据。
3.  **右键改状态**: 强制重置冷却、强制触发掉落、强制刷新配置。

### 3. 集成测试 (Integration Testing)
**场景**: 验证与 PlaceholderAPI, Vault, MythicMobs 的交互。
**原则**:
*   使用 `ServiceManager` 注册 Mock 服务提供者。
*   验证插件在依赖缺失时是否优雅降级（不报错）。

---

## 模块范围 (Module Scope) (When to Invoke)

- **User**: "帮我写一个升级系统的 Manager" -> **Action**: 同时生成 `LevelManagerTest.java`。
- **User**: "修复金币不扣除的 Bug" -> **Action**: 写测试复现 -> 修复 -> 验证。
- **User**: "实现一个复杂的蓄力技能" -> **Action**: 同时实现一个指令来显示当前蓄力值。