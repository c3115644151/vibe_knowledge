---
name: Native Blocking Pattern (1.21+)
type: pattern
---
# 问题 (Problem)
在 Minecraft 1.21+ 中，`Shield` (盾牌) 物品的机制是硬编码的。开发者经常希望创建自定义武器（如大剑或太刀），使其能够进行格挡/弹反并具有视觉上的“格挡”动作，但**不**希望真正使用盾牌物品（因为这会强制使用副手或破坏模型），同时也**不**希望失去剑的横扫攻击或其他属性。

# 解决方案 (Solution)
使用 1.21 新增的数据组件：`consumable` 和 `blocks_attacks`。
- `consumable`: 配置 `animation: 'block'` 允许任何物品在右键点击时触发格挡动作（举手）。
- `blocks_attacks`: 允许物品原生减少伤害，模拟盾牌的功能而无需真正成为盾牌。

# 用法 (Usage)
通过 NBT 注入组件（使用 `Unsafe.modifyItemStack` 或 `Paper` API 如果可用）。

```java
// NBT 构建示例
String components = """
    minecraft:iron_sword[
        blocks_attacks={
            damage_reductions:[
                {base:0, factor:0.5} // 50% 伤害减免
            ]
        },
        consumable={
            consume_seconds: 72000, // 无限持续时间 (实际上)
            animation: 'block',     // 触发盾牌格挡动画
            has_consume_particles: false,
            can_always_use: true    // 允许随时右键使用
        }
    ]
""";

// 通过 Bukkit Unsafe 应用 (Spigot/Paper)
ItemStack item = new ItemStack(Material.IRON_SWORD);
item = Bukkit.getUnsafe().modifyItemStack(item, components);

// 注意：在注入后使用 ItemMeta 应用显示名称 / Lore / 自定义模型数据
// 以确保它们被正确格式化且不会被覆盖。
```

# 优势 (Benefits)
1.  **视觉效果 (Visuals)**: 为任何物品提供真实的“格挡”姿态（举手/交叉）。
2.  **机制保留 (Mechanics)**: 保留基础物品的属性（例如剑的横扫攻击、斧头的破盾），如果只是简单地重绘盾牌材质则会丢失这些属性。
3.  **原生支持 (Native Support)**: 客户端预测完美工作；不需要延迟的抓包 Hack。

# 注意事项 (Caveats)
- `Bukkit.getUnsafe().modifyItemStack` 是已弃用的内部方法。请谨慎使用，或在稳定后切换到 Paper 的 DataComponent API。
- `consumable` 组件在某种意义上使物品变得“可食用”，但通过设置极高的 `consume_seconds`，它永远不会真正完成消耗。
