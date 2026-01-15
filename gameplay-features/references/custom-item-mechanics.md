# 自定义物品机制指南 (Custom Item Mechanics)

在开发基于原版材质的高级自定义物品（如煎锅、法杖、枪械）时，我们需要处理两个核心问题：**“如何屏蔽原版行为”**（输入控制）和**“如何管理状态与反馈”**（输出与持久化）。

本指南整合了 **原版行为屏蔽 (Behavior Suppression)** 与 **状态管理 (State Management)** 的最佳实践，通过“煎锅”案例展示完整的开发闭环。

---

## 第一部分：行为控制 (Interaction Control)

继承原版材质（如斧头、锄头）能带来良好的视觉与动作体验，但必须屏蔽不符合自定义逻辑的原版行为（如剥皮、耕地、误食副手食物）。

### 1.1 核心原则：Explicit Deny
- **精准打击**：使用 `Event.Result.DENY` 而非简单的 `setCancelled(true)`。
- **意图明确**：`DENY` 显式告知客户端“不要使用手中的物品”，这比单纯取消事件更有效，尤其是涉及副手交互时。

### 1.2 常见场景解决方案

#### 场景 A：防止工具误操作（剥皮/耕地）
当使用 `AXE` 或 `HOE` 作为材质时，右键方块会触发原版修改世界行为。

```java
@EventHandler
public void onInteract(PlayerInteractEvent event) {
    if (event.getAction() == Action.RIGHT_CLICK_BLOCK && isMyItem(event.getItem())) {
        Block block = event.getClickedBlock();
        if (Tag.LOGS.isTagged(block.getType())) {
            // 仅拒绝物品使用，允许方块交互（如开箱子）
            event.setUseItemInHand(Event.Result.DENY);
        }
    }
}
```

#### 场景 B：防止副手误食/误用（关键）
当主手持自定义物品，副手持食物/盾牌时，右键往往优先触发副手行为。

**解决方案**：如果执行了自定义逻辑，必须**强制拒绝**物品使用。

```java
if (executeCustomLogic(player, mainHandItem)) {
    // 关键：告诉客户端“本次交互已被处理，不要尝试吃副手的肉”
    event.setUseItemInHand(Event.Result.DENY);
    event.setCancelled(true); 
}
```

#### 场景 C：防止合成与修复
自定义物品不应参与原版合成或铁砧修复。

```java
@EventHandler
public void onPrepareAnvil(PrepareAnvilEvent event) {
    if (isMyItem(event.getInventory().getItem(0)) || isMyItem(event.getInventory().getItem(1))) {
        event.setResult(null); // 禁止铁砧操作
    }
}
```

---

## 第二部分：状态管理与反馈 (State & Feedback)

自定义物品通常拥有动态状态（如烹饪进度、充能点数）。**严禁**使用 Lore 存储这些数据。

### 2.1 数据存储：PersistentDataContainer (PDC)
使用 Bukkit 原生 PDC (NBT) 存储数据。
- **优势**：隐形、持久、不破坏物品堆叠（只要逻辑处理得当）。

```java
// 写入数据
public void setProgress(ItemStack item, int progress) {
    ItemMeta meta = item.getItemMeta();
    meta.getPersistentDataContainer().set(KEY_PROGRESS, PersistentDataType.INTEGER, progress);
    item.setItemMeta(meta);
}
```

### 2.2 状态反馈：ActionBar (MiniMessage)
对于高频变化的状态（进度条、冷却），使用 ActionBar 而非 Lore。
- **优势**：无库存闪烁，UI 清爽。

```java
player.sendActionBar(MiniMessage.miniMessage().deserialize(
    "<yellow>充能中: <gold>" + current + "/" + max
));
```

### 2.3 状态同步陷阱
修改 `ItemStack` 的 PDC 后，必须确保写回背包，否则在某些事件流中数据会丢失。

```java
// 1. 修改 Meta
item.setItemMeta(meta);

// 2. 强制同步回背包 (针对手持物品)
if (hand == EquipmentSlot.HAND) {
    player.getInventory().setItemInMainHand(item);
} else {
    player.getInventory().setItemInOffHand(item);
}
```

---

## 第三部分：完整案例分析——煎锅 (The Skillet)

**需求**：
1.  **材质**：`IRON_AXE` (外观)。
2.  **交互**：右键热源烹饪，**不能**给原木剥皮，**不能**误吃副手食物。
3.  **状态**：记录烹饪进度，通过 ActionBar 显示。

**综合实现**：

```java
@EventHandler
public void onInteract(PlayerInteractEvent event) {
    ItemStack item = event.getItem();
    if (!isSkillet(item)) return;

    // 1. 行为控制：防止剥皮 (针对 Block 交互)
    if (event.getAction() == Action.RIGHT_CLICK_BLOCK && Tag.LOGS.isTagged(event.getClickedBlock().getType())) {
        event.setUseItemInHand(Event.Result.DENY);
    }

    // 2. 自定义逻辑：手持烹饪
    boolean logicExecuted = false;
    if (event.getAction().isRightClick()) {
        if (tryStartCooking(event.getPlayer(), item)) {
            logicExecuted = true;
        }
    }

    // 3. 行为控制：防止副手误食 (核心兜底)
    if (logicExecuted) {
        event.setUseItemInHand(Event.Result.DENY);
        event.setCancelled(true);
    }
}

public boolean tryStartCooking(Player player, ItemStack skillet) {
    // ... 检查热源 ...
    
    // 4. 状态管理：更新 PDC
    ItemMeta meta = skillet.getItemMeta();
    meta.getPersistentDataContainer().set(KEY_IS_COOKING, PersistentDataType.BYTE, (byte) 1);
    skillet.setItemMeta(meta);
    
    // 5. 状态同步
    player.getInventory().setItemInMainHand(skillet);
    
    // 6. 反馈：启动 Task 发送 ActionBar
    startCookingTask(player);
    return true;
}
```
