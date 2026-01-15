# CraftEngine 配置深度参考 (CraftEngine Configuration Reference)

本文档作为 CraftEngine (CE) 的内部红宝书，详细记录了从基础物品到进阶自定义方块行为的配置规范，旨在避免重复造轮子和逻辑误判。

## 1. 基础配置 (Basic Configuration)

核心定义：物品与方块的基础属性、模型与掉落。

**文件路径范式**: 
- 方块: `resources/<namespace>/configuration/blocks/<category>/<block_id>.yml`
- 物品: `resources/<namespace>/configuration/items/<category>/<item_id>.yml`

### 方块配置 (Block)

```yaml
blocks:
  default:your_block:
    settings:
      # 引用模板简化配置 (推荐)
      # 位于 templates/block_settings.yml
      template:
        - default:hardness/stone
        - default:sound/stone
      # 覆盖原版属性
      overrides:
        map-color: 24
        push-reaction: DESTROY
        is-suffocating: true       # 是否会导致窒息
        is-redstone-conductor: true # 是否传导红石
    state:
      auto-state: solid            # 自动状态机类型
      model:
        template: default:model/cube_all
        arguments:
          texture: minecraft:block/custom/your_texture
```

### 物品配置 (Item)

**注意**: 物品配置必须扁平化（不要嵌套在 `item:` 下），且名称/Lore 必须位于 `data` 节点。

```yaml
items:
  namespace:item_id:
    material: PAPER
    custom-model-data: 10001
    model: namespace:item/model_file  # 可选，指向 assets/.../models/item/model_file.json
    data:
      item-name: "<!i>物品名称"     # 必须放在 data 下，支持 MiniMessage
      lore:                        # 必须放在 data 下
        - "<gray>描述行1"
        - "<gray>描述行2"
```

---

## 2. 延伸配置 (Extended Configuration)

核心定义：自定义方块状态 (Block States) 与属性映射。
解决 **"方块变种"** 和 **"视觉状态切换"** 的核心机制。

**关键字段**: `properties` (属性定义) 与 `variants` (变种映射)。

### 案例：为方块添加“含水(waterlogged)”与“朝向(facing)”支持

```yaml
    states:
      properties:
        # 定义自定义属性
        waterlogged:
          type: boolean
          default: false
        facing:
          type: horizontal_direction # 内置类型：north, east, south, west
          default: north
      
      appearances:
        # 定义视觉外观模板
        default_model:
          auto-state: solid
          model: { ... }
        waterlogged_model:
          auto-state: waterlogged_solid
          model: { ... }

      variants:
        # 属性组合 -> 视觉/设置映射
        waterlogged=false:
          appearance: default_model
        waterlogged=true:
          appearance: waterlogged_model
          settings:
            fluid-state: water    # 关键：让方块内充满水
            burnable: false       # 变种特有属性
```

**排查 Tip**: 配置修改后，必须检查 CE 生成的缓存文件 `custom-block-states.json` 是否更新，否则修改可能不生效。

---

## 3. 进阶配置 (Advanced Configuration)

核心定义：自定义行为 (Behaviors) 与 事件系统 (Event System)。

### A. 自定义行为 (Behaviors)
无需写代码即可复用复杂的逻辑模块。

```yaml
    behaviors:
      # 允许液体流过 (解决自定义方块"锁水"问题的核心行为)
      # 配合 properties: waterlogged 使用效果最佳
      - type: liquid_flowable_block
      
      # 作物行为
      - type: crop_block
        grow-speed: 0.25
        light-requirement: 9
        is-bone-meal-target: true
        bone-meal-age-bonus: 1
      
      # 灌木行为 (检查底部方块依赖)
      - type: bush_block
        bottom-blocks:
          - minecraft:grass_block
          - minecraft:dirt
```

### B. 事件系统 (Events)
在配置中直接编写交互逻辑，替代简单的 Java 监听器。

```yaml
    events:
      # 破坏事件
      - on: break
        conditions:
          - type: match_block_property
            properties:
              age: 2  # 仅在成熟时触发
        functions:
          - type: particle
            particle: minecraft:end_rod
            count: 15
          - type: play_sound
            sound: minecraft:entity.experience_orb.pickup
            
      # 右键交互事件 (例：右键收割并补种)
      - on: right_click
        conditions:
          - type: match_block_property
            properties:
              age: 7
        functions:
          - type: break_block
          - type: place_block
            block-state: default:your_crop[age=0]
          - type: drop_loot
            loot-table: default:loot_table/your_crop
          - type: swing_hand
```

---

## 4. 常见问题速查 (Troubleshooting)

| 问题现象 | 可能原因 | 解决方案 |
| :--- | :--- | :--- |
| **方块锁水** (无法含水) | 未定义 `waterlogged` 属性 | 在 `properties` 添加 `waterlogged: boolean` 并在 `variants` 中设置 `fluid-state: water`。 |
| **流体被阻断** (水流无法流过) | 缺少行为定义 | 在 `behaviors` 列表添加 `liquid_flowable_block`。 |
| **模型紫黑** | 路径错误或模板参数缺失 | 检查 `model.template` 和 `arguments` 路径是否匹配 `assets` 目录结构。 |
| **事件不触发** | 条件 (`conditions`) 未满足 | 检查 `match_block_property` 的数值类型 (int vs string) 是否匹配。 |
