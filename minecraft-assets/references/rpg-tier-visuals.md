# RPG Tier Visual Design Pattern

**核心理念**: 
通过统一的视觉语言（颜色与符号）区分物品的品质（Tier/Star），同时针对“原版通用物品”与“特产稀有物品”采用不同的展示策略。
此设计参考了 **CuisineFarming_Project (耕食记)** 与 **PastureSong_Project (牧野之歌)** 的成熟视觉体系。

## 1. 视觉分类策略

### A. 变质物品 (Variable Quality / Vanilla)
**定义**: 具有不同品质的原版物品或通用产物。
**示例**: 原矿 (Raw Iron), 作物 (Wheat), 畜牧产品 (Wool)。

*   **物品名颜色**: **动态变化** (随星级改变)。
*   **星级符号**: **动态变化** (随星级改变)。
*   **视觉效果**: 物品名称直接反映其品质等级。

> **举例**: 
> *   <green>★☆☆☆☆ 粗铁</green> (1星 - 绿色)
> *   <gold>★★★★★ 粗铁</gold> (5星 - 金色)

### B. 定质物品 (Fixed Quality / Specialty)
**定义**: 具有固有稀有度的特产或特殊物品，虽然也有品质之分，但其本身即代表某种“稀有类别”。
**示例**: 特产矿 (Lignite), 稀有作物 (Jade), 传说装备。

*   **物品名颜色**: **固定统一** (通常为金色或该类别的代表色)。
*   **星级符号**: **动态变化** (随星级改变)。
*   **视觉效果**: 强调物品本身的特殊性，品质作为附加属性展示。

> **举例**: 
> *   <green>★☆☆☆☆</green> <gold>褐煤</gold> (1星 - 名字保持金色)
> *   <gold>★★★★★</gold> <gold>褐煤</gold> (5星 - 名字保持金色)

## 2. 标准色板 (StarryForge Standard)

| 星级 | 颜色代码 | Adventure API | 含义 |
| :--- | :--- | :--- | :--- |
| 1 ★ | Green | `NamedTextColor.GREEN` | 普通 / Common |
| 2 ★ | Aqua | `NamedTextColor.AQUA` | 优秀 / Uncommon |
| 3 ★ | Light Purple | `NamedTextColor.LIGHT_PURPLE` | 稀有 / Rare |
| 4 ★ | Yellow | `NamedTextColor.YELLOW` | 史诗 / Epic |
| 5 ★ | Gold | `NamedTextColor.GOLD` | 传说 / Legendary |

## 3. 实现指南 (Java)

```java
// 伪代码示例
public void applyVisuals(ItemStack item, int stars) {
    NamedTextColor starColor = getStarColor(stars);
    Component starLore = getStarComponent(stars); // e.g., <green>★★☆☆☆
    
    ItemMeta meta = item.getItemMeta();
    List<Component> lore = meta.lore();
    lore.add(0, starLore); // 始终添加星级 Lore
    
    if (isVanillaItem(item.getType())) {
        // A类: 原版物品 -> 名字变色
        Component name = meta.displayName() != null ? meta.displayName() : Component.translatable(item.getType());
        meta.displayName(name.color(starColor));
    } else {
        // B类: 特产物品 -> 名字颜色不变 (由 items.yml 定义)
        // 仅 Lore 变色
    }
    
    item.setItemMeta(meta);
}
```
