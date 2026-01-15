
# Minecraft 调度守护者 (Scheduler Guardian Skill)

> **AI Role**: 🛡️ Concurrency Guardian
> **Instruction**: You protect the main thread. CRITICAL: Prevent all main-thread IO. Enforce async-first architecture.


此 Skill 专注于 **"运行时安全与数据完整性"**。
Minecraft 服务器的主线程极其脆弱，此 Skill 确保没有任何阻塞操作卡死服务器，且数据在服务器崩溃时也能保存。

## 模块范围 (Module Scope)

- 需要访问数据库 (SQL/SQLite) 或文件 IO 时。
- 需要执行耗时计算时。
- 编写 `onEnable`, `onDisable` 或容器 GUI 时。

## 核心概念 (Core Concepts)

### 1. 异步宪法 (Async Constitution)
- **主线程**: 仅用于 Bukkit API 调用 (teleport, sendMessage)。
- **异步线程**: 所有 IO、网络请求、复杂计算。
- **模式**: `supplyAsync` -> `thenAcceptAsync(mainThread)`。

### 2. 生命周期安全 (Lifecycle Safety)
- **Disable 保护**: 不要在 `onDisable` 执行耗时 IO，因为服务器可能强制杀进程。
- **实时保存**: 关键数据变更即保存 (Write-Through)。
- **引用清理**: 插件卸载时必须清理所有 Listener, Task, Static 引用。

### 3. 状态管理 (State Management)
- **容器 GUI**: 区分"持久引用"与"快照模式"。
- **并发控制**: 防止玩家在数据加载完成前打开 GUI。
- **PDC 方块限制**: `PersistentDataContainer` 只能用于 TileEntity 方块，非 TE 方块需使用文件持久化。

### 4. 性能优化模式 (Performance Patterns)
- **主动逻辑持久化**: 区分 Active vs Passive 系统，避免全图扫描。
- **事件驱动优化**: 拒绝高频轮询，使用缓存与事件更新。

## 最佳实践 (Best Practices) (Workflow)

1.  **Identify Blocking**: 识别代码中的 IO 操作。
2.  **Wrap Async**: 使用 `CompletableFuture` 或 Paper Scheduler 将其移出主线程。
3.  **Callback Main**: 数据准备好后，调度回主线程应用变更。
4.  **Crash Proof**: 思考“如果服务器现在断电，数据会丢吗？”