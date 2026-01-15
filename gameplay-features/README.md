
# Minecraft 特性工程师 (Feature Engineer Skill)

> **AI Role**: 🎮 Gameplay Engineer
> **Instruction**: You implement game mechanics. Focus on engaging, bug-free, and performant gameplay logic.


此 Skill 专注于 **"如何实现具体的游戏机制"**。
它是 `mc-mechanic-researcher` (理论) 的实践者，负责将理论转化为代码。

## 模块范围 (Module Scope)

- 制作自定义物品 (Custom Items) 时。
- 需要在世界中显示全息图或模型 (Display Entities) 时。
- 构建多方块机器 (Multiblocks) 时。

## 核心概念 (Core Concepts)

### 1. 高级实体管理 (Display Entities)
- **Display API**: 优先使用 Display Entity (Text/Item/Block) 代替 ArmorStand。
- **变换**: 使用 Transformation 矩阵进行旋转、缩放。

### 2. 自定义物品 (Custom Items)
- **PDC**: 使用 `PersistentDataContainer` 存储物品 ID 和属性，严禁依赖 Lore。
- **ItemMeta**: 正确处理 ItemFlag 和属性修饰符。

### 3. 多方块结构 (Multiblocks)
- **配置驱动**: 使用字符矩阵定义结构。
- **自动旋转**: 实现基于朝向的相对坐标检测。

### 4. 复杂农业 (Advanced Farming)
- **双格作物**: 处理上下联动、含水状态与碰撞箱问题。

## 最佳实践 (Best Practices) (Workflow)

1.  **Define Data**: 定义物品或实体的 NBT/PDC 数据结构。
2.  **Implement Logic**: 编写生成、更新、销毁的逻辑。
3.  **Optimize**: 考虑大量实体或方块更新时的性能影响。