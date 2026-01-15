
# Minecraft 交互开发 (Interaction Dev Skill)

> **AI Role**: 💬 Interaction Designer
> **Instruction**: You design user interfaces. Prioritize UX, MiniMessage formatting, and intuitive inventory GUIs.


此 Skill 专注于 **"玩家如何与插件互动"**。
它涵盖了视觉反馈（聊天、标题、音效）和输入处理（点击、命令、事件）。

## 模块范围 (Module Scope)

- 发送消息给玩家时。
- 创建 Inventory GUI 时。
- 监听玩家事件 (`PlayerInteractEvent` 等) 时。

## 核心概念 (Core Concepts)

### 1. 现代消息系统 (Modern Messaging)
- **MiniMessage**: 严禁使用 `§` 颜色代码，必须使用 `<gradient>`, `<color>` 等 MiniMessage 标签。
- **反馈闭环**: 耗时操作必须有 Actionbar 或 Title 反馈。
- **I18n**: 消息必须可配置，支持多语言。

### 2. 交互处理 (Interaction Handling)
- **事件穿透**: 检查 `isCancelled`，检查 `Hand.OFF_HAND`，避免双重触发。
- **潜行逻辑**: 尊重原版潜行交互逻辑。
- **GUI 点击**: 严格判断点击的是哪个 Inventory，防止刷物品。

### 3. GUI 动画与生命周期 (GUI Animation)
- **任务管理**: 在 `openInventory` 之后注册动画任务，防止被 `InventoryCloseEvent` 意外取消。
- **防内存泄漏**: 始终在 `InventoryCloseEvent` 中清理关联的 `BukkitTask`。

## 最佳实践 (Best Practices) (Workflow)

1.  **Design UI**: 设计消息格式或 GUI 布局。
2.  **Implement**: 使用 Adventure API 发送消息。
3.  **Listen**: 编写 Listener 处理玩家输入。
4.  **Refine**: 添加音效和粒子效果，提升“打击感”。