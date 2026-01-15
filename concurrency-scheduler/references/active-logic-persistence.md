# 主动逻辑持久化 (Active Logic Persistence)

## 核心概念

Minecraft 中的方块逻辑分为两种主要驱动模式：**被动（Passive）** 和 **主动（Active）**。

### 1. 被动模式 (Passive / Random Tick)
- **机制**: 依赖原版 `is-randomly-ticking: true`。Minecraft 每 tick 随机抽取 chunk 中的方块进行更新。
- **数据存储**: 状态（如生长阶段）直接存储在 Chunk Data (BlockState) 中。
- **优点**: 零内存占用，符合原版直觉。
- **缺点**: 无法精确控制时间（完全随机），无法脱离 Chunk 加载运行。
- **适用场景**: 普通农作物、随缘生长的植物。

### 2. 主动模式 (Active / Scheduled)
- **机制**: 插件使用 Scheduler (`runTaskTimer`) 主动驱动逻辑。
- **挑战**: 为了驱动逻辑，插件必须知道**哪些位置**有目标方块。
- **内存注册表**: 需要一个 `Set<Location>` 或 `Map<Location, Data>` 来维护活跃对象。
- **持久化需求**: 
  - **问题**: 内存数据在重启后丢失。
  - **陷阱**: 如果尝试在 `onChunkLoad` 时扫描整个 Chunk 寻找方块来重建注册表，会导致 O(N) 的巨大性能开销（全图扫描）。
  - **解决方案**: **必须**将注册表独立持久化（如 `crops.yml` 或数据库）。

## 最佳实践

当实现自定义生长逻辑（如现实时间生长、定点刷新）时：

1.  **独立存储**: 使用 YAML/JSON/SQL 存储所有活跃方块的坐标。
2.  **启动加载**: 服务器启动时一次性将坐标加载到内存注册表 (`RiceManager`).
3.  **增量更新**: 玩家种植/破坏时，同步更新内存和文件（或异步写入数据库）。
4.  **区块无关性**: 逻辑运行不依赖区块是否加载（或者在逻辑运行时判断区块加载状态，未加载则跳过，避免强制加载区块导致的卡顿）。

### 代码示范 (Code Example)

```java
public class CropManager {
    // 内存注册表：只存储活跃方块坐标
    private final Set<Location> activeCrops = ConcurrentHashMap.newKeySet();
    private final File file;

    // 启动时加载：O(1) 磁盘读取，不触碰 World API
    public void load() {
        if (!file.exists()) return;
        YamlConfiguration config = YamlConfiguration.loadConfiguration(file);
        List<String> stored = config.getStringList("locations");
        stored.forEach(s -> activeCrops.add(Serializer.deserializeLoc(s)));
    }

    // 运行时逻辑：只遍历内存表，不遍历 Chunk
    public void tick() {
        Iterator<Location> it = activeCrops.iterator();
        while (it.hasNext()) {
            Location loc = it.next();
            // 性能关键：先检查区块是否加载，避免强制加载未加载区块
            if (!loc.getWorld().isChunkLoaded(loc.getBlockX() >> 4, loc.getBlockZ() >> 4)) {
                continue; 
            }
            
            // 执行生长逻辑
            if (growLogic(loc)) {
                it.remove(); // 生长结束，移出活跃列表
                saveAsync();
            }
        }
    }
}
```

## 反模式 (Anti-Pattern)

```java
// ❌ 错误做法：依赖区块加载事件扫描方块
@EventHandler
public void onChunkLoad(ChunkLoadEvent event) {
    for (BlockState tile : event.getChunk().getTileEntities()) {
        // 极慢！且不包含非 TileEntity 方块
    }
    // 或者更糟：遍历 Chunk 内所有 Block (65536个)
}
```
