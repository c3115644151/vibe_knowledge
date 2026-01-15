# 向量数学优化 (Vector Math Optimization)

## 核心问题

在判断"玩家是否看着某个物体"时，直觉通常是使用射线追踪（RayTracing）。

- **RayTrace**: `player.rayTraceBlocks(distance)`
- **成本**: 极高。需要遍历视线上的每一个方块，进行碰撞箱检测。
- **场景**: 如果你需要每 tick 检查所有玩家是否看着某个全息图（TextDisplay）来切换显示内容，RayTrace 会导致 TPS 骤降。

## 反模式 (Anti-Pattern)

```java
// ❌ 错误做法：在 tick 循环中使用 RayTrace
public void tick() {
    for (Player player : Bukkit.getOnlinePlayers()) {
        // 性能杀手：rayTraceBlocks 会进行复杂的物理碰撞箱计算
        // 且随着距离增加，计算量线性增长
        RayTraceResult result = player.rayTraceBlocks(10.0);
        
        if (result != null && result.getHitBlock().getType() == Material.DIAMOND_BLOCK) {
            player.sendMessage("Rich!");
        }
    }
}
```

## 解决方案：向量点积 (Dot Product)

利用向量点积计算视线向量与目标向量的夹角。

### 原理
- **A**: 玩家视线向量 (`player.getEyeLocation().getDirection()`)
- **B**: 玩家眼睛到目标的向量 (`targetPos - eyePos`)
- **Dot**: `A · B.normalize()`
- **结果**: 
  - `1.0`: 正视。
  - `0.0`: 垂直。
  - `-1.0`: 背对。

### 代码实现

```java
// ✅ 极速判断玩家是否看着目标（约 25度角锥）
public boolean isLookingAt(Player player, Location target, double threshold) {
    Vector eye = player.getEyeLocation().toVector();
    Vector toTarget = target.toVector().subtract(eye);
    
    // 距离检查（先剔除太远的）
    if (toTarget.lengthSquared() > 64) return false; // > 8 blocks

    Vector lookDir = player.getEyeLocation().getDirection();
    double dot = lookDir.dot(toTarget.normalize());

    return dot > threshold; // 推荐 0.95
}
```

## 性能对比
- **RayTrace**: ~5000ns / op (随距离增加)
- **Dot Product**: ~5ns / op (常数级)
- **提升**: **1000倍**

## 适用场景
- 全息图（Hologram）交互提示。
- 视线触发的机关。
- 怪物仇恨判定（是否在正面扇形区域）。
