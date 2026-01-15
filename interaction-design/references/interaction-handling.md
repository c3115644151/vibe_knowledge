# 交互处理规范 (Interaction Handling)

## 1. 交互去重 (Interaction Debouncing)
**问题**: Minecraft 客户端/服务端通信机制导致以下情况经常发生：
1.  **放置连击**: 放置方块后的瞬间，客户端可能判定鼠标未抬起，再次发送右键交互包，导致刚放下的方块立即被交互。
2.  **主副手双发**: 主手交互完成后，服务端/客户端可能尝试触发副手交互（尤其是在主手事件被取消或特定返回时）。
3.  **快速点击**: 玩家无意的高频点击。

**强制规范**:
必须引入 `InteractionDebouncer` (基于 UUID 的冷却机制) 来过滤同一 Tick 或短时间内的重复交互。

```java
// 工具类
public class InteractionDebouncer {
    private static final Map<UUID, Long> lastInteract = new ConcurrentHashMap<>();
    
    public static boolean canInteract(UUID uuid) {
        long now = System.currentTimeMillis();
        if (now - lastInteract.getOrDefault(uuid, 0L) < 50) return false; // 50ms 冷却
        lastInteract.put(uuid, now);
        return true;
    }
    
    // 在放置方块时手动阻塞交互
    public static void blockInteraction(UUID uuid, long duration) {
        lastInteract.put(uuid, System.currentTimeMillis() + duration);
    }

    // 重置冷却 (Yield 模式专用)
    public static void reset(UUID uuid) {
        lastInteract.remove(uuid);
    }
}

// Listener 实现
@EventHandler
public void onInteract(PlayerInteractEvent event) {
    // 1. 去重检查
    if (!InteractionDebouncer.canInteract(event.getPlayer().getUniqueId())) {
        event.setCancelled(true); 
        return;
    }
    
    // 2. 业务逻辑
    boolean handled = manager.handleInteract(event.getPlayer(), ...);
    
    if (handled) {
        event.setCancelled(true);
    } else {
        // 3. 关键：Yield 模式 -> 重置去重器 -> 允许副手事件通过
        InteractionDebouncer.reset(event.getPlayer().getUniqueId());
    }
}
```

## 2. 放置与交互的互斥 (Placement-Interaction Mutex)
当玩家放置一个带有自定义交互功能的方块时，必须确保**放置动作本身**不会触发该方块的交互逻辑。

**规范**:
1.  在 `BlockPlaceEvent` (或 `CustomBlockPlaceEvent`) 中，调用 `InteractionDebouncer.blockInteraction(uuid, 200)` 给玩家施加约 200ms (4 ticks) 的交互冷却。
2.  这能有效防止“放置砧板 -> 瞬间把手里的第二个砧板放上去”的问题。

## 3. 主副手逻辑与状态原子性
在处理 `PlayerInteractEvent` 时，不要假设主手处理完就结束了。

**场景**: 主手拿刀切肉。
**错误逻辑**: 主手切肉 -> 砧板变空 -> 副手(拿肉)触发交互 -> 砧板放入肉。
**结果**: 玩家觉得肉刚切好又自动放进去了。

**规范**:
1.  **原子性**: 如果主手触发了有效交互（如切菜、取物），则该次点击事件应被视为“完全消费”。
2.  **去重**: 上述的 `InteractionDebouncer` 会自动解决此问题，因为主手交互会更新 `lastInteractTime`，导致几毫秒后的副手交互被 `canInteract` 拦截。

## 4. 事件穿透与让路 (Event Passthrough & Yield)
当主手物品无法与方块交互（例如：主手持剑右键空砧板）时，应允许副手物品（例如：副手持肉）尝试交互。

**Yield & Reset 模式**:
1.  **Manager 层**: `handleInteract` 返回 `false` 表示“未处理/让路”。
2.  **Listener 层**: 接收到 `false` 后，调用 `InteractionDebouncer.reset(uuid)`。
3.  **原理**: 撤销主手交互产生的冷却，使紧随其后的副手交互包（在同一 Tick 或极短时间内到达）能通过 `canInteract` 检查。

## 5. 实体显示与持久化
关于 `ItemDisplay` 或 `ArmorStand` 等视觉实体的持久化管理（如防止幽灵方块、生命周期管理），请移步查阅：
👉 [display-entity-management.md](./display-entity-management.md)
