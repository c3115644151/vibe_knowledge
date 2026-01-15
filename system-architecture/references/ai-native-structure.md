# AI-Native 目录结构规范 (Feature-First)

## 1. 核心理念：上下文共现性 (Context Co-location)

AI (LLM) 与人类开发者在阅读代码时有本质区别：
- **人类**: 习惯“按层分包” (Package by Layer)，例如 `commands/`, `listeners/`, `managers/`。因为人类的大脑构建了抽象的心理模型，可以跨文件夹联想。
- **AI**: 依赖**上下文窗口 (Context Window)**。如果一个功能的实现分散在 5 个不同的文件夹中，AI 就需要同时检索并加载这 5 个文件才能完整理解逻辑。这会导致：
    1.  **Token 浪费**: 加载大量无关的“邻居文件”。
    2.  **幻觉风险**: 跨文件推断逻辑时容易出错。
    3.  **修改遗漏**: 忘记修改某个偏远文件夹里的关联代码。

因此，**AI 原生**的目录结构必须遵循 **"Package by Feature" (按功能分包)** 原则。

---

## 2. 推荐结构 (Recommended Structure)

### ❌ 传统结构 (Package by Layer) - 对 AI 不友好
```text
src/main/java/com/example/plugin/
├── commands/
│   ├── BanCommand.java
│   ├── WarpCommand.java
│   └── ShopCommand.java
├── listeners/
│   ├── PlayerJoinListener.java
│   ├── BlockBreakListener.java
│   └── ShopClickListener.java
├── managers/
│   ├── BanManager.java
│   ├── WarpManager.java
│   └── ShopManager.java
└── utils/
    └── ...
```
*问题：要修改“商店”功能，AI 需要跨越 `commands`, `listeners`, `managers` 三个目录，容易漏改。*

### ✅ AI 原生结构 (Package by Feature) - 强烈推荐
```text
src/main/java/com/example/plugin/
├── Core.java (Main class, 极简)
├── api/ (公共接口)
├── features/ (核心功能模块)
│   ├── ban/
│   │   ├── BanCommand.java
│   │   ├── BanListener.java
│   │   ├── BanManager.java
│   │   └── BanStorage.java
│   ├── warp/
│   │   ├── WarpCommand.java
│   │   ├── WarpManager.java
│   │   └── WarpData.java
│   └── shop/
│       ├── ShopCommand.java
│       ├── ShopGUI.java (Listener inside or separate)
│       └── ShopConfig.java
└── utils/ (真正通用的工具，如 StringUtil)
```
*优势：修改“商店”功能时，AI 只需要关注 `features/shop/` 目录，所有相关逻辑尽收眼底。*

---

## 3. 模块化标准 (The Module Pattern)

为了进一步降低 AI 的认知负担，每个 Feature 应该是一个**自包含的模块**。

建议创建一个基础接口 `GameFeature` 或 `GameModule`：

```java
public interface GameFeature {
    void onEnable();
    void onDisable();
    // 可选：自动注册逻辑
    default void registerCommands() {}
    default void registerListeners() {}
}
```

每个功能包下有一个入口类（例如 `ShopFeature.java`），负责组装该模块的所有组件。主类 (`Core.java`) 只需要注册这个 Feature，而不需要关心它内部有多少个 Listener。

```java
// features/shop/ShopFeature.java
public class ShopFeature implements GameFeature {
    @Override
    public void onEnable() {
        var manager = new ShopManager();
        registerCommand(new ShopCommand(manager));
        registerListener(new ShopListener(manager));
    }
}
```

## 4. 实施规则

1.  **新建功能时**: 必须在 `features/` 下创建新包。
2.  **重构旧代码时**: 遇到修改需求，优先将分散的代码聚合到同一个 Feature 包中。
3.  **可见性控制**: 
    - Feature 内部的实现细节（如 Listener, Manager 实现）应设为 `package-private`。
    - 仅暴露必要的接口或 Facade 类。
4.  **反模式**:
    - 严禁创建巨大的 `utils` 包（除非是真正的通用 String/Math 工具）。
    - 严禁创建 `listeners` 包（Listener 必须跟随 Feature）。
