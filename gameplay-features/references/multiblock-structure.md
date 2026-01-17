# 多方块结构 (Multiblock Structure) 微框架

## 总纲 (General Outline)

本框架旨在解决 Minecraft 多方块结构开发中的**定义繁琐**、**旋转适配困难**和**调试不便**三大痛点。通过引入“字符矩阵定义”与“相对坐标系”，开发者只需在一个 3D 字符数组中“画”出结构，即可自动获得任意方向的检测能力。

### 核心架构
1.  **配置驱动 (Config Driven)**: 结构形状在 `config.yml` 中使用字符矩阵定义，实现数据与逻辑分离。
2.  **模式对象 (StructurePattern)**: 运行时将配置解析为不可变对象，包含层级数据 (Layers)、匹配器 (Matchers) 和核心偏移 (CoreOffset)。
3.  **智能检测 (Smart Detection)**:
    -   **相对坐标**: 所有检测基于核心方块（Core Block）的相对位置。
    -   **自动旋转**: 只需定义标准朝向（北），算法自动处理东南西三个方向的旋转变换。
4.  **全息调试 (Holographic Debug)**: 独有的 `analyze` 方法可返回所有不匹配方块的精确位置与预期类型，便于生成粒子提示。

### 适用场景
- **工业机器**: 如高炉、粉碎机等固定结构。
- **魔法祭坛**: 需要特定方块摆放的仪式结构。
- **大型设施**: 传送门框架、防御塔等。

---

## 核心特性
1.  **字符矩阵定义**: 使用类似 ASCII Art 的方式直观定义结构层。
2.  **自动旋转**: 只需定义一次（默认朝北），即可自动支持 4 个方向的检测。
3.  **相对坐标**: 基于核心方块（Core Block）的相对坐标系，无需硬编码绝对位置。
4.  **调试友好**: 提供 `analyze` 方法返回所有不匹配的方块，便于给玩家提供“全息投影”式的错误提示。

## 代码实现

### 1. StructurePattern 类
核心模式定义类。

```java
public class StructurePattern {
    private final String[][] layers; // [y][z] -> x
    private final Map<Character, Predicate<Block>> matchers = new HashMap<>();
    private final Vector coreOffset; // 核心方块在模式中的位置

    // ... 构造函数 ...

    public StructurePattern addMatcher(char symbol, Material material) { ... }

    // 检查结构是否完整
    public boolean check(Block coreBlock, BlockFace facing) { ... }

    // 分析结构差异（用于调试/提示）
    public Map<Block, String> analyze(Block coreBlock, BlockFace facing) { ... }
}
```

### 2. 定义示例 (Alloy Blast Furnace)

```java
// 3x3x3 结构
// C: 深板岩瓦, P: 磨制深板岩, M: 岩浆块, B: 高炉(核心), S: 锻造台
String[][] layers = {
    // Layer 1 (Bottom)
    { "CPC", "PMP", "CPC" },
    // Layer 2 (Middle)
    { "CPC", "PBP", "CSC" },
    // Layer 3 (Top)
    { "CPC", "PCP", "CPC" }
};

// 定义模式，核心方块(B)位于 (1, 1, 1)
StructurePattern pattern = new StructurePattern(layers, new Vector(1, 1, 1))
    .addMatcher('C', Material.DEEPSLATE_TILES)
    .addMatcher('P', Material.POLISHED_DEEPSLATE)
    .addMatcher('M', Material.MAGMA_BLOCK)
    .addMatcher('B', Material.BLAST_FURNACE)
    .addMatcher('S', Material.SMITHING_TABLE)
    .addMatcher(' ', Material.AIR);
```

### 3. 检测逻辑

```java
// 尝试 4 个方向检测
BlockFace[] faces = {BlockFace.NORTH, BlockFace.EAST, BlockFace.SOUTH, BlockFace.WEST};
for (BlockFace face : faces) {
    if (pattern.check(coreBlock, face)) {
        return face; // 找到结构，返回朝向
    }
}
```

## 最佳实践
1.  **核心方块**: 选择一个具有交互功能的方块作为核心（如高炉、箱子），所有检测都相对于它进行。
2.  **空方块**: 明确定义空气 `' '`，确保结构内部不会被杂物填充（如果需要）。
3.  **用户反馈**: 检测失败时，使用 `analyze` 方法获取错误列表，并用粒子效果高亮显示缺失/错误的方块位置。

## 适用场景
- 自定义机器 (Custom Machines)
- 祭坛/仪式结构 (Rituals)
- 传送门框架 (Portal Frames)

---

## 特殊案例：微型多方块与根节点归一化 (Micro-Multiblock & Root Normalization)

在处理双格高作物（如水稻、玉米）等“微型多方块结构”时，常遇到的一个严重问题是**交互错位**。

### 问题描述 (The Problem)
点击或交互双高作物的**上半部分**时，逻辑往往错误地将其视为独立的方块，导致：
1.  **第三层 Bug (Third Layer Bug)**: 对上半部分使用骨粉，逻辑误以为它是根部，在其上方生成新的方块，导致作物悬空或出现第三层。
2.  **生长停滞 (Stuck Growth)**: 逻辑检查上半部分的 `crop_stage`（通常为 0 或未定义），而非根部的实际阶段，导致生长条件判断失败。

### 解决方案 (The Solution)
在执行任何逻辑（生长、破坏、交互）之前，**强制将坐标归一化到根方块**（通常是下半部分）。这确保了状态的“单一真理来源” (Single Source of Truth)。

### 代码模式 (Code Pattern)
在所有交互/生长方法的入口处添加归一化逻辑：

```java
public void handleInteraction(Location loc) {
    Block block = loc.getBlock();

    // CraftEngine 集成检查
    if (com.example.simplecuisine.util.CraftEngineHook.isEnabled()) {
        // 获取 'half' 属性: "0"=下半部, "1"=上半部
        String half = com.example.simplecuisine.util.CraftEngineHook.getCustomBlockProperty(block, "half");
        
        // 如果是上半部，向下偏移至根部
        if ("1".equals(half)) {
            block = block.getRelative(BlockFace.DOWN);
        }
    }
    
    // 此时 'block' 保证是根部 (Lower) 方块
    int stage = getRiceStage(block); // 安全地查询状态
    // ... 执行逻辑 ...
}
```

### 适用性
- **CraftEngine 双格方块**: 必须应用此模式。
- **原版双格花卉**: 类似的逻辑（虽然原版通常自己处理，但在自定义逻辑中需手动处理）。
