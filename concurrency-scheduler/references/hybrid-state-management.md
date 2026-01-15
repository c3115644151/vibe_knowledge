# 容器方块状态管理双雄 (The Dual Patterns of Container State Management)

> **前言**: 在 Bukkit 开发中，处理带有 GUI 交互、物品消耗、后台运行的机器方块（如熔炉、洗矿台、烹饪锅）是公认的深水区。
> 本文档基于实战复盘（StarryForge 洗矿台与 SimpleCuisine 烹饪锅），提炼出两种截然不同但各有所长的架构模式。

## 🏛️ 模式一：持久引用模式 (The Persistent Reference Pattern)

> **代表作**: SimpleCuisine - CookingPot
> **核心哲学**: "只有一个真理 (The One True Inventory)"

### 1. 核心机制
在机器初始化（区块加载）时，直接获取并**永久持有** TileEntity 的 `Inventory` 对象引用。所有的逻辑操作（Tick、GUI 交互、输入输出）都直接作用于这个同一个内存对象。

```java
public class Machine {
    // 关键：持久持有 Inventory 引用
    private final Inventory inventory; 
    
    public Machine(Location loc) {
        // 在构造时获取一次，之后不再调用 block.getState()
        if (loc.getBlock().getState() instanceof Container container) {
            this.inventory = container.getInventory();
        }
    }

    public void tick() {
        // 直接修改引用，无需 update()
        // Bukkit/NMS 会自动同步这个 Inventory 到 TileEntity 和玩家界面
        inventory.setItem(0, newItem); 
    }
}
```

### 2. 优势 (Pros)
*   **✅ 极简代码**: 不需要手动调用 `block.update()`，不需要处理 Snapshot 回写。
*   **✅ 天然同步**: 无论是后台逻辑修改，还是玩家 GUI 操作，改的都是同一个对象，不存在并发冲突。
*   **✅ 高性能**: 避免了每秒调用 `getState()` (涉及 NBT 解析) 和 `update()` (涉及区块重绘/光照更新) 的开销。
*   **✅ 动画流畅**: 因为没有 `block.update()` 导致的 TileEntity 重置，GUI 动画极其丝滑。

### 3. 劣势 (Cons)
*   **⚠️ 引用失效风险**: 如果区块卸载后重载，旧的 `Inventory` 引用可能会失效（指向一个已经不存在的 TileEntity），导致操作无效或内存泄漏。需要配合严谨的 `ChunkLoad/Unload` 监听来重建对象。
*   **⚠️ 依赖 NMS 实现**: 这种模式依赖于 Bukkit `CraftInventory` 背后的 NMS `TileEntity` 引用是稳定的。虽然在主流版本（1.16+）中表现良好，但属于隐式行为。

---

## 🛡️ 模式二：强制快照模式 (The Force Refresh Pattern)

> **代表作**: StarryForge - SluiceManager (Hybrid 2.0)
> **核心哲学**: "原子化事务 (Atomic Transactions)"

### 1. 核心机制
不持有任何长期引用。每次需要修改数据（消耗/产出）时，强制从世界获取最新的 `BlockState` (Snapshot)，修改后立即写回。视觉动画（倒计时）与逻辑数据分离。

```java
public class MachineManager {
    // 只有数据 Session，没有 Inventory 引用
    private final Map<Location, Session> sessions = new HashMap<>();

    public void tick() {
        // 1. 视觉层：仅修改玩家打开的 LiveView (不存盘)
        if (hasViewer) updateGui(player.getOpenInventory());
        
        // 2. 事务层：需要修改物品时
        if (needsTransaction) {
            // 🚨 强制获取最新快照
            BlockState freshState = loc.getBlock().getState(); 
            if (freshState instanceof Container container) {
                // 写入数据
                container.getInventory().addItem(result);
                // 强制提交
                freshState.update(true); 
            }
        }
    }
}
```

### 2. 优势 (Pros)
*   **✅ 绝对健壮**: 不依赖任何内存引用的生命周期。无论区块怎么卸载/重载，每次 `getState()` 拿到的肯定是最新的。
*   **✅ 解耦**: 逻辑层 (Session) 与 表现层 (Inventory) 完全分离，适合跨区块的大型系统管理。
*   **✅ 兼容性强**: 不依赖 NMS 行为，纯标准 Bukkit API 操作，跨版本兼容性理论上更好。

### 3. 劣势 (Cons)
*   **⚠️ 性能开销**: 频繁的 `getState()` 和 `update()` 有 IO 和 CPU 成本。
*   **⚠️ 复杂度高**: 必须小心翼翼地分离“视觉更新”和“事务更新”。一旦在视觉更新时错误地调用了 `update()`，就会导致 GUI 闪烁或物品回滚（即我们遭遇的 4 小时 Bug）。

---

## ⚖️ 决策指南 (Decision Matrix)

| 场景 | 推荐模式 | 理由 |
| :--- | :--- | :--- |
| **单方块机器 (熔炉/锅/箱子)** | **🏆 持久引用模式** | 代码最简单，Bug 最少，性能最好。SimpleCuisine 证明了其稳定性。 |
| **多方块结构 / 复杂网络** | **🛡️ 强制快照模式** | 当一个逻辑对应多个方块，或者方块可能动态变化时，快照模式更安全。 |
| **极高频 GUI 动画 (每 tick)** | **🏆 持久引用模式** | `update()` 造成的网络包和客户端重置会让快照模式难以驾驭。 |
| **新手/不熟悉生命周期** | **🛡️ 强制快照模式** | 虽然代码啰嗦，但不容易出现“改了没反应”这种引用失效的诡异问题。 |

## 🧠 总结
*   如果你的机器逻辑简单，且生命周期管理得当（能正确处理 Chunk 加载/卸载），**请优先使用持久引用模式**。这是最优雅的解法。
*   如果你发现引用经常失效，或者你需要处理非常复杂的跨区块逻辑，**强制快照模式**是你的兜底方案。但切记：**视觉层别碰 BlockState，事务层必须拿新 BlockState。**
