# 示例：现代消息与交互
展示如何使用 MiniMessage 和 Title API。

```java
package com.example.demo;

import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.minimessage.MiniMessage;
import net.kyori.adventure.title.Title;
import org.bukkit.entity.Player;

import java.time.Duration;

public class MessageUtil {
    private static final MiniMessage mm = MiniMessage.miniMessage();

    public static void sendSuccess(Player player, String message) {
        // ❌ 旧写法: player.sendMessage("§a" + message);
        // ✅ 新写法
        player.sendMessage(mm.deserialize("<green>✔ " + message));
        
        // 伴随音效
        player.playSound(player.getLocation(), org.bukkit.Sound.BLOCK_NOTE_BLOCK_PLING, 1f, 2f);
    }

    public static void showLevelUp(Player player, int level) {
        Component title = mm.deserialize("<gradient:gold:yellow>LEVEL UP!</gradient>");
        Component subtitle = mm.deserialize("<gray>你达到了等级 <gold>" + level);
        
        Title.Times times = Title.Times.times(Duration.ofMillis(500), Duration.ofMillis(3000), Duration.ofMillis(1000));
        player.showTitle(Title.title(title, subtitle, times));
    }
}
```
