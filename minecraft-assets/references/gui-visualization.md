# 示例：GUI 可视化设计
在编写 Inventory 代码前，先在注释或设计文档中“画”出界面。

## 场景：酿酒锅 (Brewing Cauldron)
这是一个 3 行 (27 slots) 的界面。

### 1. ASCII 布局图
```text
+-----------------------------+
| [B] [B] [B] [B] [I] [B] [B] [B] [B] |  Row 0
| [B] [W] [F] [F] [M] [F] [F] [E] [B] |  Row 1
| [B] [B] [B] [B] [S] [B] [B] [B] [B] |  Row 2
+-----------------------------+

图例 (Legend):
[B] = Border (Gray Stained Glass Pane) - 装饰边框
[I] = Info Book (Knowledge Book)       - 帮助信息
[W] = Water Slot (Bucket)              - 水位指示
[F] = Fuel Slot (Blaze Powder)         - 燃料槽
[M] = Main Ingredient (Middle)         - 核心材料
[E] = Empty Bottle Slot                - 放空瓶的地方
[S] = Start Button (Green Wool)        - 开始酿造
```

### 2. 生成的代码 (Java)
```java
public class BrewingGUI {
    private static final int SLOT_INFO = 4;
    private static final int SLOT_WATER = 10;
    private static final int SLOT_MAIN = 13;
    private static final int SLOT_START = 22;
    // ...
    
    public void open(Player player) {
        Inventory inv = Bukkit.createInventory(null, 27, Component.text("酿酒锅"));
        
        // 1. 填充边框 [B]
        ItemStack border = new ItemStack(Material.GRAY_STAINED_GLASS_PANE);
        int[] borderSlots = {0,1,2,3,5,6,7,8, 9,17, 18,19,20,21,23,24,25,26};
        for (int slot : borderSlots) {
            inv.setItem(slot, border);
        }
        
        // 2. 设置功能槽
        inv.setItem(SLOT_START, getStartButton());
        // ...
    }
}
```
