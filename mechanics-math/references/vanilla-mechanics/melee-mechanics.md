---
name: Vanilla Melee Mechanics
type: mechanics
source: https://zh.minecraft.wiki/w/%E8%BF%91%E6%88%98%E6%94%BB%E5%87%BB
---

# Minecraft Java Edition 近战攻击机制详解 (Deep Dive: Melee Mechanics)

> **版本适用性**: Java Edition 1.21+
> **用途**: 核心战斗系统开发参考、伤害计算还原、自定义武器行为设计。

## 1. 攻击结算流程 (Attack Resolution Lifecycle)

当玩家发起一次近战攻击时，服务器按以下顺序执行逻辑：

1.  **前置检查**: 检查冷却时间、距离、视线等。
2.  **伤害计算**: 计算最终伤害值 (Final Damage)。
3.  **实际伤害应用**: 调用 `entity.damage()`。
    *   如果攻击被取消 (Event Cancelled) 或无敌帧阻挡，流程终止。
    *   **破盾判定**: 如果目标格挡成功，触发破盾逻辑（如斧头）。
4.  **后处理效果 (Post-Hit Effects)**:
    *   **击退 (Knockback)**: 应用物理冲量。
    *   **横扫 (Sweep)**: 对周围敌人造成溅射伤害。
    *   **附魔效果**: 火焰附加 (Fire Aspect)、节肢杀手 (Bane of Arthropods) 等。
    *   **反伤**: 计算荆棘 (Thorns) 或守护者尖刺。
5.  **反馈**: 播放音效、生成粒子、增加疲劳度 (Exhaustion)。

---

## 2. 伤害计算公式 (Damage Math)

最终伤害并非简单的加乘，而是分层计算。

$$
\text{Final} = (\text{Base} + \text{Item} + \text{Effects}) \times \text{Cooldown} \times \text{Crit} + \text{Enchant} \times \text{Cooldown} + \text{Extra}
$$

### A. 基础层 (Base Layer)
*   **玩家基础**: 1.0
*   **物品属性**: `generic.attack_damage` (如钻石剑 +7)。
*   **状态效果**:
    *   **力量 (Strength)**: `Level * 3.0`
    *   **虚弱 (Weakness)**: `Level * -4.0` (最低归零)

### B. 冷却修正 (Cooldown Factor)
由 `generic.attack_speed` 决定。
*   **因子 $C$**:
    *   $C = 0.2 + 0.8 \times (\frac{t}{T})^2$
    *   $t$: 距离上次攻击的 ticks。
    *   $T$: 满蓄力所需 ticks ($20 / \text{AttackSpeed}$)。
*   **影响**: 作用于 **基础层** 和 **附魔层**，但不影响 **额外伤害(重锤)**。

### C. 暴击修正 (Critical Multiplier)
*   **倍率**: $1.5\times$
*   **作用范围**: 仅作用于 **基础层** (Base + Item + Effects)。
*   **注意**: 在 Java 版中，**附魔伤害(锋利等) 不享受暴击加成**。

### D. 附魔层 (Enchantment Layer)
*   **锋利 (Sharpness)**: `0.5 * Level + 0.5`
*   **亡灵杀手/节肢杀手**: `2.5 * Level`
*   **特性**: 此伤害不被暴击放大，但受冷却惩罚影响。

### E. 额外伤害 (Extra Damage - Heavy Core)
仅适用于 **重锤 (Mace)**。
*   **公式**: 基于下落高度 $h$ (blocks)。
    *   $1.5 < h \le 3$: $4h$
    *   $3 < h \le 8$: $2h + 6$
    *   $h > 8$: $h + 14$
*   **致密附魔 (Density)**: 每级增加 $0.5 \times h$。
*   **特性**: 不受冷却影响，但受暴击加成影响（存疑，需实测，Wiki表述较模糊，通常视为独立加成）。

---

## 3. 判定与范围 (Targeting & Range)

### 攻击范围 (Reach)
*   **属性**: `player.entity_interaction_range` (1.20.5+)。
*   **计算**:
    *   **玩家**: 视线 Raytrace 距离（默认为 3.0，创造模式 5.0）。
    *   **目标**: 目标的 **受击包围盒 (Hitbox)** 会向外膨胀 `target_width + 2.04 - 0.6` (约 0.828 格) 用于判定。这意味着大体型生物更容易被击中。

### 视线检查
除玩家外，所有生物攻击前必须通过 `canSee(target)` 检查（基于 Raytrace，防止穿墙）。

---

## 4. 特殊机制详解

### A. 暴击 (Critical Hit)
*   **条件 (AND)**:
    1.  冷却 > 84.8%
    2.  `fallDistance > 0` (下落中)
    3.  不接触地面、不在梯子/藤蔓/水中
    4.  无失明效果、无骑乘
    5.  **非疾跑状态**
*   **效果**: 基础伤害 $\times 1.5$，播放 `crit` 粒子和音效。

### B. 疾跑击退 (Sprint Knockback)
*   **条件**: 玩家处于疾跑状态。
*   **效果**:
    *   **不会暴击**。
    *   造成额外的高额击退。
    *   攻击后强制停止疾跑。

### C. 横扫攻击 (Sweep Attack)
*   **条件 (AND)**:
    1.  冷却 > 84.8%
    2.  站在地面上 (与暴击互斥)
    3.  非疾跑
    4.  移动速度较低
    5.  主手是 **剑**
*   **范围**: 目标周围 1.0 格膨胀盒 + 上下 0.25 格。
*   **伤害公式**:
    *   基础: $1.0$
    *   属性加成: `generic.sweeping_damage_ratio` (比率 $R$)
    *   最终横扫伤害: $1.0 + R \times \text{BaseDamage}$
*   **附魔 (Sweeping Edge)**: 提升 $R$ 值。
    *   I: 50%, II: 67%, III: 75%。

### D. 盾牌禁用 (Shield Disabling)
*   **触发者**: 斧头 (Axes) 或 卫道士/监守者。
*   **机制**: 攻击时有概率（基于效率附魔和随机）使目标盾牌进入冷却。
*   **冷却**: 100 ticks (5秒)。

---

## 5. 击退力学 (Knockback Mechanics)

击退是一个矢量计算。
*   **方向**: 攻击者视线方向 (Yaw)。
*   **力度 (Strength)**:
    *   默认为 0。
    *   **击退附魔 (Knockback)**: 每级 +1。
    *   **疾跑**: +1。
*   **抗性 (Resistance)**: `generic.knockback_resistance` (0.0 - 1.0)。
    *   直接按比例减少击退水平速度。
    *   如果抗性为 1.0，完全免疫。
*   **垂直击退**:
    *   对非玩家实体：直接施加 0.4 m/s 向上速度（如果位于地面）。

---

## 6. 音效反馈 (Audio Feedback)

Minecraft 根据攻击结果播放不同音效，这是打击感的关键来源。

| 情景 | 音效 ID | 条件 |
| :--- | :--- | :--- |
| **暴击** | `entity.player.attack.crit` | 触发暴击条件 |
| **强击 (Strong)** | `entity.player.attack.strong` | 冷却 > 84.8% 且造成伤害 |
| **横扫** | `entity.player.attack.sweep` | 触发横扫条件 |
| **击退** | `entity.player.attack.knockback` | 触发疾跑击退 |
| **弱击 (Weak)** | `entity.player.attack.weak` | 冷却 < 84.8% |
| **无伤 (No Dmg)** | `entity.player.attack.nodamage` | 攻击无效 (如友军伤害关闭) |

---

## 7. 常见开发陷阱 (Pitfalls)

1.  **无敌帧冲突**: 高频攻击（如 1 tick 1 次）会被目标的 10 tick 无敌帧吞掉大部分伤害。需手动设置 `entity.setNoDamageTicks(0)` 才能实现高频伤害。
2.  **暴击与跳劈**: 暴击仅由**下落状态**决定。如果不跳跃（如下落台阶），也能触发暴击；反之，跳跃上升阶段攻击不会暴击。
3.  **属性覆盖**: 修改 `generic.attack_damage` 会导致原版武器属性失效（如剑不再有默认伤害），必须同时手动设置攻速。
