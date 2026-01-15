# 示例：国际化消息管理

## 1. 语言文件 (src/main/resources/lang/zh_CN.yml)
```yaml
messages:
  prefix: "<gradient:gold:yellow>[CuisineFarming]</gradient> "
  errors:
    no-permission: "<red>你没有权限执行此操作 (<gray><permission><red>)。"
    player-only: "<red>只有玩家可以使用此命令。"
  farming:
    planted: "<green>成功种植了 <gold><crop><green>。"
    harvested: "<green>收获了 <gold><amount> <gray>x <white><crop>!"
```

## 2. Java 消息工具 (Lang.java)
```java
public class Lang {
    private static FileConfiguration langConfig;
    private static final MiniMessage mm = MiniMessage.miniMessage();

    public static void init(Plugin plugin) {
        // 自动保存并加载 zh_CN.yml
        File file = new File(plugin.getDataFolder(), "lang/zh_CN.yml");
        if (!file.exists()) {
            plugin.saveResource("lang/zh_CN.yml", false);
        }
        langConfig = YamlConfiguration.loadConfiguration(file);
    }

    public static void send(CommandSender sender, String key, Placeholder... placeholders) {
        String template = langConfig.getString("messages." + key);
        if (template == null) {
            sender.sendMessage(Component.text("Missing lang key: " + key));
            return;
        }
        
        // 处理前缀
        String prefix = langConfig.getString("messages.prefix", "");
        
        // 解析 MiniMessage
        TagResolver[] resolvers = Arrays.stream(placeholders)
            .map(p -> Placeholder.component(p.key, Component.text(p.value)))
            .toArray(TagResolver[]::new);
            
        Component message = mm.deserialize(prefix + template, resolvers);
        sender.sendMessage(message);
    }
    
    public record Placeholder(String key, String value) {}
}

// 使用示例
Lang.send(player, "farming.harvested", 
    new Lang.Placeholder("amount", "5"),
    new Lang.Placeholder("crop", "番茄")
);
```
