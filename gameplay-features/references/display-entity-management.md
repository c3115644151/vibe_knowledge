# 实体显示 (Display Entity) 管理最佳实践

在现代 Minecraft 插件开发中，`ItemDisplay`, `TextDisplay`, `BlockDisplay` 等实体被广泛用于视觉效果。
然而，管理这些实体的生命周期（Lifecycle）极具挑战性，尤其是涉及服务器重启、区块卸载/加载时，容易产生“幽灵方块”（Ghost Entities）或“视觉丢失”问题。

本指南详细说明了如何正确管理持久化显示实体。

## 核心原则

1.  **持久化 (Persistence)**: 必须设置 `entity.setPersistent(true)`。
    -   **原因**: 防止区块卸载或服务器重启时实体消失。
    -   **误区**: 设置为 `false` 虽然能避免“幽灵方块”，但会导致区块重新加载时视觉效果丢失，用户体验极差。

2.  **唯一标识 (Identification)**: 必须使用 `ScoreboardTags` 或 `PersistentDataContainer` 标记你的实体。
    -   **代码**: `entity.addScoreboardTag("myplugin_visual_id");`
    -   **作用**: 允许你在未来精确地检索和清理它们。

3.  **状态锚点 (State Anchor)**: 必须有一个持久化的数据源（YAML, Database, 或 Block PDC）记录“这里应该有什么”。
    -   **逻辑**: 实体只是数据的视觉投影。数据源才是真理。
    -   **空状态保存**: 在保存数据时，**不要跳过空方块**。如果一个位置被占用（即使当前无物品），它仍需被保存，否则重启后插件会遗忘该位置，导致交互失效。

4.  **延迟恢复 (Delayed Restoration)**: 在 `ChunkLoadEvent` 中恢复视觉时，必须延迟 1 tick。
    -   **原因**: `getNearbyEntities` 在区块刚加载的一瞬间可能无法检索到所有实体，导致清理逻辑失效，进而产生重叠的重复实体。

## 标准实现模式

### 1. 创建实体
```java
public void createVisual(Location loc, ItemStack item) {
    ItemDisplay display = loc.getWorld().spawn(loc, ItemDisplay.class);
    display.setItemStack(item);
    
    // 关键：标记实体
    display.addScoreboardTag("myplugin_visual");
    
    // 关键：设置为持久化
    display.setPersistent(true); 
}
```

### 2. 清理实体
在生成新实体前，**必须**清理旧实体。
```java
public void cleanupVisualsAt(Location loc) {
    if (loc == null || loc.getWorld() == null) return;
    
    // 搜索范围略大于实体大小
    for (Entity e : loc.getWorld().getNearbyEntities(loc, 1.0, 1.0, 1.0)) {
        // 仅清理带有特定标签的实体
        if (e.getScoreboardTags().contains("myplugin_visual")) {
            e.remove();
        }
    }
}
```

### 3. 区块加载恢复 (ChunkLoad Logic)
这是防止幽灵方块和重叠的关键。
```java
@EventHandler
public void onChunkLoad(ChunkLoadEvent event) {
    // 关键：延迟 1 tick 执行
    Bukkit.getScheduler().runTaskLater(plugin, () -> {
        if (!event.getChunk().isLoaded()) return;
        
        // 你的逻辑：检查该区块内是否有需要恢复的视觉效果
        plugin.getVisualManager().restoreVisualsInChunk(event.getChunk());
    }, 1L);
}
```

### 4. 恢复逻辑 (Restore Logic)
```java
public void restoreVisualsInChunk(Chunk chunk) {
    // 1. 从数据源（如 Map 或 YAML）查找该区块内的所有目标位置
    Set<Location> targetLocations = getTargetsInChunk(chunk);
    
    for (Location loc : targetLocations) {
        // 2. 先清理旧的（防止重复）
        cleanupVisualsAt(loc);
        
        // 3. 再创建新的
        createVisual(loc, getData(loc));
    }
}
```

### 5. 服务器生命周期管理 (Server Lifecycle)
除了区块加载，服务器的启动和关闭也是关键节点。

*   **Disable (关闭/重载)**:
    1.  遍历内存中所有活跃的自定义方块管理器。
    2.  执行 `save()` 持久化数据（务必包含空容器状态）。
    3.  执行 `removeAllVisuals()` 清理所有 ItemDisplay 实体。
    4.  **目的**: 保证存档干净，不残留实体，防止下次启动时的 UUID 冲突或幽灵方块。

*   **Enable (启动)**:
    1.  读取配置文件/数据库。
    2.  执行 `restoreVisuals()` 重建所有实体。

## 常见问题排查

| 现象 | 原因 | 解决方案 |
| :--- | :--- | :--- |
| **实体消失** | `persistent` 设为了 `false`，区块卸载后实体被删。 | 设置 `setPersistent(true)`。 |
| **实体重叠 (变亮/闪烁)** | `restoreVisuals` 时未能成功清理旧实体。通常是因为直接在 `ChunkLoadEvent` 中调用 `getNearbyEntities`，此时实体尚未完全加载。 | 使用 `runTaskLater(..., 1L)` 延迟执行恢复逻辑。 |
| **服务器重启后变成无法交互的幽灵** | 实体保留了，但你的插件内存数据（Map）丢失了，导致插件不知道那是它的实体。 | 确保插件启动时加载数据源 (`loadFromYaml`)，并能够通过 Tag 识别并控制这些实体。 |
