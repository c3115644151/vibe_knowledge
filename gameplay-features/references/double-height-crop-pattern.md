# 双格作物最佳实践 (Double-Height Crop Pattern)

## Context
在实现类似水稻 (Rice)、玉米或向日葵等高度超过一格的自定义作物时，需要处理复杂的上下半部联动、碰撞箱、含水状态以及 CraftEngine 配置限制。

## Anti-Patterns (反模式) - 不要这样做！

### 1. 错误的碰撞箱配置
**错误做法**: 试图在配置中强行关闭碰撞箱。
```yaml
overrides:
  collision: false # ❌ 错误：CraftEngine 不支持此属性，会导致配置解析失败
```
**后果**: 插件报错，方块无法放置。
**修正**: 选择天然无碰撞的 `auto-state` (如 `TRIPWIRE`, `LEAVES`, `SEAGRASS`)。

### 2. 错误的枚举类型属性
**错误做法**: 使用 `enum` 类型定义方块状态。
```yaml
properties:
  half:
    type: enum # ❌ 错误：可能导致解析异常 "invalid type parameter"
    options: [lower, upper]
```
**后果**: 注册失败。
**修正**: 使用 `int` (0/1) 或 `string` 代替枚举。

### 3. 上下一致的 Auto-State
**错误做法**: 上半部分和下半部分使用相同的 `auto-state`。
```yaml
# 上半部分在空气中，却使用了含水方块
appearances:
  crop_stage=3,half=1:
    auto-state: waterlogged_leaves # ❌ 错误：会导致空中出现水流
```
**后果**: 作物上半部分凭空流水，视觉效果极差。
**修正**: 根据位置区分 `auto-state`。下半部分用 `waterlogged_...`，上半部分用 `tripwire` 或 `leaves` (不含水)。

### 4. 忽略泥土退化
**错误做法**: 仅关注作物本身，忽略其下方的耕地/泥土。
**后果**: 原版机制会导致水下或阴暗处的泥土变成草方块，破坏农田景观。
**修正**: 监听 `BlockFormEvent` 和 `BlockSpreadEvent`，保护作物下方的方块。

## Best Practice (最佳实践) - 这样做！

### 1. 分离的 Auto-State 策略
利用 `half` 属性区分上下半部，分别映射到最合适的原版方块状态。

```yaml
states:
  properties:
    crop_stage: {type: int, range: 0~3}
    half: {type: int, range: 0~1} # 0=Lower, 1=Upper
  appearances:
    # 下半部分 (水中): 使用含水叶子
    crop_stage=3,half=0:
      auto-state: waterlogged_tintable_leaves
      model: my:rice_lower
    # 上半部分 (空气): 使用高绊线 (无水、无碰撞)
    crop_stage=3,half=1:
      auto-state: higher_tripwire
      model: my:rice_upper
```

### 2. 智能属性降级
优先使用基础类型 (`int`, `string`) 而非复杂类型，确保跨版本兼容性。

### 3. 完整的生命周期管理
不要依赖原版生长机制 (RandomTick)，而是实现主动管理：
*   **种植**: 检查水源深度、下方土壤。
*   **生长**: 定时任务扫描 -> 概率生长 -> 检查空间 -> 更新方块状态。
*   **收割**: 破坏上半部分 -> 掉落成品 -> 下半部分重置为 Stage 0 (保留根部)。
*   **清除**: 破坏下半部分 -> 掉落种子 -> 清除整株 -> **恢复水源** (防止水被置换为空气)。

### 4. 鲁棒的破坏逻辑
```java
public void handleBreak(Location loc) {
    // 1. 识别当前部分 (Top/Bottom)
    // 2. 联动破坏另一半
    // 3. 恢复环境:
    //    - 如果下方是泥土，恢复为 WATER (防止干旱)
    //    - 否则恢复为 AIR
}
```
