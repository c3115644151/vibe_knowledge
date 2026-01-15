# PDC 方块类型限制 (PersistentDataContainer Block Type Limitations)

> **警示**: 这是一个极易陷入的**经验主义陷阱**。当你从一个成功的 PDC 持久化实现复制到另一个功能时，必须首先验证目标方块的类型。

## 核心问题

`PersistentDataContainer` (PDC) **只能用于 TileEntity 方块**。非 TileEntity 方块的 `BlockState` 不是 `TileState` 的实例，因此没有 PDC 可用。

### TileEntity 方块示例 (可用 PDC)
| 方块 | 用途 |
|------|------|
| `BLAST_FURNACE` | 合金锻炉核心 ✅ |
| `BARREL` | 洗矿机 ✅ |
| `CHEST` / `TRAPPED_CHEST` | 存储 ✅ |
| `FURNACE` / `SMOKER` | 熔炼 ✅ |
| `HOPPER` | 物品传输 ✅ |
| `BREWING_STAND` | 酿造 ✅ |
| `BEACON` | 信标 ✅ |
| `LECTERN` | 讲台 ✅ |
| `SIGN` (各种告示牌) | 文本存储 ✅ |

### 非 TileEntity 方块示例 (无法使用 PDC)
| 方块 | 常见误用场景 |
|------|-------------|
| `SMITHING_TABLE` | 星魂共振台核心 ❌ |
| `CRAFTING_TABLE` | 工作台 ❌ |
| `STONECUTTER` | 切石机 ❌ |
| `GRINDSTONE` | 砂轮 ❌ |
| `CARTOGRAPHY_TABLE` | 制图台 ❌ |
| `LOOM` | 织布机 ❌ |
| `ANVIL` | 铁砧 ❌ |
| 大多数装饰方块 | 墙/栅栏/楼梯等 ❌ |

## 陷阱案例：StarryForge 星魂台

### 背景
合金锻炉使用 `BLAST_FURNACE` 作为核心，成功实现了基于 PDC 的会话持久化。开发者试图将相同的模式应用于星魂台（使用 `SMITHING_TABLE` 作为核心）。

### 失败代码
```java
public void saveToBlock() {
    Block block = anvilLocation.getBlock();
    // 🚨 SMITHING_TABLE 不是 TileState，这个条件永远为 false！
    if (!(block.getState() instanceof TileState state)) {
        return; // 静默失败，数据从未保存
    }

    PersistentDataContainer pdc = state.getPersistentDataContainer();
    pdc.set(Keys.ALTAR_BLUEPRINT, PersistentDataType.STRING, data);
    state.update();
}
```

### 问题
- 代码**静默失败** - 没有任何错误日志
- 多次"修复"都失败 - 因为根本问题未被识别
- 开发者困惑：为什么完全相同的模式在合金锻炉成功但在星魂台失败？

### 根因
| 系统 | 核心方块 | 是否 TileEntity | PDC 可用 |
|------|---------|----------------|---------|
| 合金锻炉 | `BLAST_FURNACE` | ✅ 是 | ✅ 可用 |
| 星魂台 | `SMITHING_TABLE` | ❌ 否 | ❌ 不可用 |

## 解决方案

### 方案 1：更换核心方块为 TileEntity
如果设计允许，将多方块结构的核心换成 TileEntity 方块。

```yaml
# 修改前
S: SMITHING_TABLE  # 无法持久化

# 修改后
S: BARREL          # 可以持久化（需要更新材质/模型）
```

### 方案 2：文件持久化 (推荐)
对于无法更换核心方块的情况，使用独立 YAML 文件存储会话数据。

```java
public class ForgingManager {
    private final File dataFile;
    private final Map<Location, ForgingSession> sessions = new HashMap<>();

    public void saveSessions() {
        YamlConfiguration config = new YamlConfiguration();
        int index = 0;
        for (Map.Entry<Location, ForgingSession> entry : sessions.entrySet()) {
            Location loc = entry.getKey();
            String key = "sessions.session_" + index;

            config.set(key + ".world", loc.getWorld().getName());
            config.set(key + ".x", loc.getBlockX());
            config.set(key + ".y", loc.getBlockY());
            config.set(key + ".z", loc.getBlockZ());
            config.set(key + ".blueprint", SerializationUtils.itemToBase64(session.getBlueprint()));
            // ... 其他数据
            index++;
        }
        config.save(dataFile);
    }

    public void loadSessions() {
        if (!dataFile.exists()) return;
        YamlConfiguration config = YamlConfiguration.loadConfiguration(dataFile);
        // 从文件恢复所有会话...
    }
}
```

### 方案 3：使用结构中的其他 TileEntity
如果多方块结构中包含 TileEntity 方块，可以将数据存储在那个方块上。

```java
// 星魂台结构: 底座有 POLISHED_DEEPSLATE (非TE)，但可以添加一个隐藏的 BARREL
public void saveToStructure() {
    // 找到结构中的 TileEntity 方块
    Block storageBlock = findTileEntityInStructure();
    if (storageBlock.getState() instanceof TileState state) {
        state.getPersistentDataContainer().set(...);
        state.update();
    }
}
```

## 防御性编程

### 必须添加的日志
```java
public void saveToBlock() {
    Block block = location.getBlock();
    if (!(block.getState() instanceof TileState state)) {
        // ✅ 关键：必须记录日志，否则问题会被隐藏
        plugin.getLogger().warning(
            "[Persistence] Block at " + location + " (" + block.getType() +
            ") is not a TileEntity! Cannot use PDC persistence."
        );
        return;
    }
    // ...
}
```

### 设计阶段检查清单
在设计多方块结构时，问自己：

1. ☐ 这个结构需要持久化状态吗？
2. ☐ 核心方块是 TileEntity 吗？（检查 `block.getState() instanceof TileState`）
3. ☐ 如果不是，我选择哪种替代方案？
   - 更换核心方块
   - 文件持久化
   - 使用结构中其他 TileEntity

## 快速验证代码

```java
// 在开发阶段验证方块类型
public static void debugBlockType(Block block) {
    BlockState state = block.getState();
    System.out.println("Block: " + block.getType());
    System.out.println("Is TileState: " + (state instanceof TileState));
    System.out.println("State class: " + state.getClass().getSimpleName());
}
```

## 总结

| 情况 | 推荐方案 |
|------|---------|
| 核心是 TileEntity | 直接使用 PDC |
| 核心非 TE，但可更换 | 更换为 TileEntity 方块 |
| 核心非 TE，不可更换 | **文件持久化** (推荐) |
| 结构中有其他 TE | 借用该 TE 的 PDC |

> **黄金法则**: 永远不要假设一个方块支持 PDC。在实现持久化之前，先用 `instanceof TileState` 验证。
