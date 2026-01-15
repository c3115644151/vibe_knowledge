# 启动竞态条件 (Startup Race Condition)

## 问题定义 (Problem Definition)

在 Minecraft 插件开发中，**依赖加载 (SoftDepend/Depend)** 仅保证插件类的**加载顺序**，**不保证**依赖插件的**数据就绪状态**。

### 典型场景
你的插件 `MyPlugin` 依赖 `ItemsAdder` 或 `CraftEngine`。
1. 服务器启动。
2. `ItemsAdder` 执行 `onEnable()` -> 启动异步任务加载 5000+ 个物品配置 -> `onEnable` 结束。
3. `MyPlugin` 执行 `onEnable()` -> 立即调用 `ItemsAdder.getAllItems()` -> 返回空列表 (因为异步任务还在跑)。
4. `MyPlugin` 报错或功能失效 (配方加载失败)。

这就是经典的 **启动时序竞态 (Startup Race Condition)**。

## 解决方案 (The Solution)

**核心原则**: 永远不要在 `onEnable` 中直接请求外部插件的动态数据。

### 1. 下一刻调度 (Next Tick Scheduling)
利用 Bukkit 调度器将任务推迟到服务器启动流程的**主循环开始时**执行。此时所有插件的 `onEnable` 都已跑完，大部分初始化工作已完成。

```java
@Override
public void onEnable() {
    // 注册指令和监听器 (这些是安全的)
    getCommand("mycmd").setExecutor(new MyCmd());

    // 危险操作：请求外部数据 -> 推迟执行
    Bukkit.getScheduler().runTask(this, () -> {
        getLogger().info("开始加载配方 (Delayed)...");
        // 此时 ItemsAdder/CraftEngine 通常已完成初始化
        loadRecipes(); 
    });
}
```

### 2. 事件驱动 (Event Driven)
如果依赖插件提供了 "Load Complete" 事件，优先监听该事件。

```java
@EventHandler
public void onItemsAdderLoaded(ItemsAdderLoadDataEvent event) {
    getLogger().info("ItemsAdder 数据已就绪，开始重载配方...");
    loadRecipes();
}
```
*注意*: 这种方式更稳健，但并非所有插件都提供此类事件。

## 最佳实践 (Best Practices)

1.  **构造函数纯净原则**: Manager/Service 类的构造函数不应包含任何“重”逻辑或外部 API 调用。只做变量初始化。
2.  **显式加载方法**: 将数据加载逻辑剥离为 `load()` 或 `init()` 方法。
3.  **防御性判空**: 即使使用了延迟加载，获取外部数据时仍需判空，并提供降级方案 (Graceful Degradation)。
