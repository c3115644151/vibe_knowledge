# Bukkit GUI Animation Lifecycle Pattern

## Context
在 Bukkit/Spigot 插件开发中，在自定义 GUI (Inventory) 中实现动态元素（如循环播放的物品动画）通常需要使用 `BukkitTask` (runTaskTimer)。

## The Problem (Anti-Pattern)
当在打开新菜单的方法中**先注册动画任务**，然后**再打开菜单**时，会导致任务立即被取消。

```java
// 错误示范
public void openMenu(Player player) {
    Inventory inv = Bukkit.createInventory(...);
    
    // 1. 注册新任务
    BukkitTask task = runAnimationTask(...);
    activeTasks.put(player.getUniqueId(), task.getTaskId());
    
    // 2. 打开菜单
    player.openInventory(inv); 
    // ^^^ 这会触发之前打开菜单的 InventoryCloseEvent
}

@EventHandler
public void onClose(InventoryCloseEvent event) {
    // CloseEvent 逻辑：清理玩家当前的任务
    if (activeTasks.containsKey(player.getUniqueId())) {
        cancelTask(activeTasks.remove(player.getUniqueId()));
    }
}
```

**原因分析**:
1. `player.openInventory(inv)` 会强制关闭玩家当前打开的 GUI。
2. 这会触发 `InventoryCloseEvent`。
3. 如果 `InventoryCloseEvent` 的处理器包含“清理该玩家所有动画任务”的逻辑，它会从 map 中移除并取消**刚刚注册的**新任务（因为 map 是按玩家 ID 索引的，新任务覆盖了旧任务 ID，或者仅仅是移除了当前 ID）。
4. 结果：新菜单打开了，但动画任务被杀死了。

## The Solution (Pattern)
**必须在 `player.openInventory(inv)` 之后注册动画任务。**

```java
// 正确示范
public void openMenu(Player player) {
    Inventory inv = Bukkit.createInventory(...);
    
    // 1. 先打开菜单
    player.openInventory(inv);
    // ^^^ 触发旧菜单的 CloseEvent，清理旧任务
    
    // 2. 再注册新任务
    BukkitTask task = runAnimationTask(...);
    activeTasks.put(player.getUniqueId(), task.getTaskId());
}
```

**逻辑流**:
1. `openInventory(inv)` 执行 -> 触发旧菜单 `CloseEvent` -> 旧任务被清理。
2. `runAnimationTask(...)` 执行 -> 新任务被创建并注册。
3. 结果：新菜单打开，新任务正常运行。

## Robustness Checklist
1. **Holder Check**: 在 Runnable 中检查 `inventory.getHolder()` 或 `view.getTopInventory()` 是否仍是预期的 GUI，防止玩家切换菜单后任务继续修改错误的 GUI。
2. **Task Cleanup**: 始终在 `InventoryCloseEvent` 中清理任务，防止内存泄漏 (Memory Leaks)。
