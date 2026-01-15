# 通用调试棒 (Debug Stick) 实现

在开发复杂的 Minecraft 插件时，游戏内的调试工具能极大地提高效率。与其反复修改代码打印日志，不如提供一个“调试棒”，通过点击交互实时查看数据或触发逻辑。

## 核心功能

1.  **物品识别**: 使用带有特殊 NBT 标签或 CustomModelData 的物品（如烈焰棒）。
2.  **信息查看 (左键)**: 点击方块或实体，发送详细的调试信息（BlockState, EntityData, PDC 数据等）。
3.  **模式切换 (Shift+右键)**: 切换不同的调试模式（例如：查看模式、修改模式、测试模式）。
4.  **功能触发 (右键)**: 执行当前模式下的调试逻辑（例如：强制生长、重置状态）。

## 参考实现 (`DebugStickManager`)

以下是一个基于 Paper API 的简单调试棒管理器实现。

```java
package com.example.plugin.debug;

import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.format.NamedTextColor;
import org.bukkit.Material;
import org.bukkit.NamespacedKey;
import org.bukkit.entity.Player;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.block.Action;
import org.bukkit.event.player.PlayerInteractEvent;
import org.bukkit.inventory.ItemStack;
import org.bukkit.inventory.meta.ItemMeta;
import org.bukkit.persistence.PersistentDataType;
import org.bukkit.plugin.java.JavaPlugin;

import java.util.ArrayList;
import java.util.List;

public class DebugStickManager implements Listener {

    private final JavaPlugin plugin;
    private final NamespacedKey debugKey;

    public DebugStickManager(JavaPlugin plugin) {
        this.plugin = plugin;
        this.debugKey = new NamespacedKey(plugin, "debug_stick");
        plugin.getServer().getPluginManager().registerEvents(this, plugin);
    }

    /**
     * 获取调试棒物品
     */
    public ItemStack getDebugStick() {
        ItemStack stick = new ItemStack(Material.BLAZE_ROD);
        ItemMeta meta = stick.getItemMeta();
        meta.displayName(Component.text("🔧 调试棒", NamedTextColor.GOLD));
        List<Component> lore = new ArrayList<>();
        lore.add(Component.text("左键: 查看目标信息", NamedTextColor.GRAY));
        lore.add(Component.text("右键: 执行调试动作", NamedTextColor.GRAY));
        meta.lore(lore);
        meta.getPersistentDataContainer().set(debugKey, PersistentDataType.BYTE, (byte) 1);
        stick.setItemMeta(meta);
        return stick;
    }

    /**
     * 给予玩家调试棒
     */
    public void giveDebugStick(Player player) {
        player.getInventory().addItem(getDebugStick());
        player.sendMessage(Component.text("已获取调试棒！", NamedTextColor.GREEN));
    }

    @EventHandler
    public void onInteract(PlayerInteractEvent event) {
        Player player = event.getPlayer();
        ItemStack item = event.getItem();

        // 1. 检查是否手持调试棒
        if (item == null || item.getType() == Material.AIR) return;
        if (!item.hasItemMeta()) return;
        if (!item.getItemMeta().getPersistentDataContainer().has(debugKey, PersistentDataType.BYTE)) return;

        // 2. 权限检查
        if (!player.hasPermission("plugin.debug")) {
            player.sendMessage(Component.text("你没有调试权限。", NamedTextColor.RED));
            return;
        }

        event.setCancelled(true); // 阻止默认行为

        // 3. 左键：查看信息
        if (event.getAction() == Action.LEFT_CLICK_BLOCK) {
            if (event.getClickedBlock() != null) {
                player.sendMessage(Component.text("=== 方块信息 ===", NamedTextColor.GOLD));
                player.sendMessage(Component.text("类型: " + event.getClickedBlock().getType()));
                player.sendMessage(Component.text("位置: " + event.getClickedBlock().getLocation().toVector()));
                player.sendMessage(Component.text("数据: " + event.getClickedBlock().getBlockDataAsString()));
                // 这里可以添加更多自定义逻辑，如读取 PDC
            }
        }
        
        // 4. 右键：执行动作 (示例：破坏方块不掉落)
        else if (event.getAction() == Action.RIGHT_CLICK_BLOCK) {
             if (event.getClickedBlock() != null) {
                 event.getClickedBlock().setType(Material.AIR);
                 player.sendMessage(Component.text("已移除方块 (Debug)", NamedTextColor.YELLOW));
             }
        }
        
        // 5. 空手右键：其他功能
        else if (event.getAction() == Action.RIGHT_CLICK_AIR) {
            player.sendMessage(Component.text("调试模式切换功能待实现...", NamedTextColor.AQUA));
        }
    }
}
```

## 集成建议

1.  **命令注册**: 添加一个 `/debugstick` 或 `/ds` 命令来调用 `giveDebugStick`。
2.  **配置开关**: 在 `config.yml` 中添加 `debug-mode: true/false`，仅在开启时注册此监听器。
3.  **扩展性**: 可以将右键行为抽象为 `DebugAction` 接口，支持多种调试模式切换。
