# 原版附魔机制 (Vanilla Enchantment Mechanisms)

> **来源**: Minecraft 官方 Wiki (zh)
> **用途**: 提供原版附魔行为的精确参考，确保插件实现符合原版逻辑。
> **维护**: 当游戏机制变更或添加新附魔时，应更新此文件。

## 1. 战斗类附魔 (Combat Enchantments)

### 1.1 抢夺 (Looting)
**命名空间 ID**: `minecraft:looting`
**最高等级**: 3
**适用物品**: 剑 (Sword)

#### 核心机制
当玩家（或驯服的狼）持有该附魔物品杀死生物时，抢夺附魔会增加掉落物的数量和概率。

#### 效果详解

1.  **常见掉落物 (Common Drops)**
    *   **效果**: 增加掉落数量的**最大值**。
    *   **规则**: 每级增加 1 个最大掉落数。
    *   **分布**: 根据等级随机决定额外掉落。
    *   **公式 (近似)**: `Count = Random(Min, Max + Level)`
    *   *注意*: 某些掉落物可能有特定的分布方式，但 "Max + Level" 是标准的通用行为。

2.  **稀有掉落物 (Rare Drops)**
    *   **效果**: 增加掉落的**概率**。
    *   **规则**: 每级增加 1% 的概率。
    *   **Wiki 数据**:
        *   基础: 2.5% (典型的稀有掉落，如僵尸掉落铁锭)
        *   等级 I: 3.5% (+1%)
        *   等级 II: 4.5% (+1%)
        *   等级 III: 5.5% (+1%)
    *   **代码范式**:
        ```java
        double chance = baseChance + (lootingLevel * 0.01);
        ```

3.  **装备掉落 (Equipment Drops)**
    *   **效果**: 增加生物掉落其穿戴装备的概率。
    *   **规则**: 每级增加 1% 的概率。
    *   **Wiki 数据 (Java版)**:
        *   基础: 8.5%
        *   等级 I: 9.5%
        *   等级 II: 10.5%
        *   等级 III: 11.5%

#### 例外情况
抢夺附魔**不**影响以下内容：
*   凋灵 (下界之星)
*   末影龙 (龙蛋)
*   铁傀儡 (铁锭/虞美人 - 固定范围/机制)
*   不死图腾 (唤魔者)
*   监守者 (Warden)

## 2. 工具类附魔 (Tool Enchantments)

### 2.1 耐久 (Unbreaking)
**命名空间 ID**: `minecraft:unbreaking`
**最高等级**: 3
**适用物品**: 工具, 武器, 盔甲

#### 核心机制
对于工具、武器和盔甲，耐久附魔（Unbreaking）的作用是减少物品在使用时消耗耐久度的概率。

#### 效果详解

1.  **工具与武器 (Tools & Weapons)**
    *   **公式**: $ P_{loss} = \frac{1}{Level + 1} $
    *   **概率表**:
        *   **Level 0**: 100% 消耗 (1/1)
        *   **Level 1**: 50% 消耗 (1/2)
        *   **Level 2**: 33.3% 消耗 (1/3)
        *   **Level 3**: 25% 消耗 (1/4)

2.  **盔甲 (Armor)**
    *   **公式**: $ P_{loss} = 60\% + \frac{40\%}{Level + 1} $
    *   **概率表**:
        *   **Level 1**: 80% 消耗
        *   **Level 2**: 73.3% 消耗
        *   **Level 3**: 70% 消耗

#### 代码范式 (Java)

```java
import org.bukkit.enchantments.Enchantment;
import org.bukkit.inventory.ItemStack;
import org.bukkit.inventory.meta.Damageable;
import java.util.concurrent.ThreadLocalRandom;

public void applyDurabilityLoss(ItemStack item, int amount) {
    if (item == null || item.getType().getMaxDurability() <= 0) return;
    
    ItemMeta meta = item.getItemMeta();
    if (!(meta instanceof Damageable)) return;
    
    Damageable damageable = (Damageable) meta;
    // 注意：Enchantment.DURABILITY 对应 minecraft:unbreaking
    int unbreakingLevel = item.getEnchantmentLevel(Enchantment.DURABILITY);
    
    // 工具/武器公式 (对于盔甲需使用另一公式)
    if (ThreadLocalRandom.current().nextInt(unbreakingLevel + 1) == 0) {
        int newDamage = damageable.getDamage() + amount;
        damageable.setDamage(newDamage);
        item.setItemMeta(damageable);
        
        if (newDamage >= item.getType().getMaxDurability()) {
            item.setAmount(0); // Break item
            // 播放破碎音效...
        }
    }
}
```
