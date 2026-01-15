# 防御性编程 (Defensive Coding)

## 1. 配置文件安全读取 (Safe Config Loading)
永远不要信任配置文件中的数据。

```java
// ❌ 危险：如果路径不存在或类型不对，可能返回 null 或默认值
String val = config.getString("path");

// ✅ 安全：始终提供默认值
String val = config.getString("path", "default_value");
int count = config.getInt("settings.max-count", 10);
```

## 2. 空值检查 (Null Checks)
Minecraft API 中充满了潜在的 null 值。

```java
// ItemMeta 检查
if (item.hasItemMeta()) {
    var meta = item.getItemMeta();
    // ...
}

// Inventory 检查 (空槽位可能是 null 而不是 AIR)
ItemStack[] contents = inventory.getContents();
for (ItemStack stack : contents) {
    if (stack == null || stack.getType().isAir()) continue;
    // ...
}

// Command 参数检查
if (args.length == 0) {
    sender.sendMessage(mm.deserialize("<red>请输入参数！"));
    return true;
}
```
