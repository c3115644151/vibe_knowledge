# Feature-Based Configuration Loading

在 Feature-First 架构下，我们依然要坚持 **"配置与代码分离"** 的原则。
虽然代码（逻辑）是模块化的，但数据（配置）依然需要存储在易于编辑的 YAML 文件中。

## 1. 错误示范 (Hardcoding)

AI 容易为了省事，直接在 Manager 里写死物品数据。这是**严禁**的。

```java
// ❌ 错误：硬编码
package com.example.plugin.features.shop;

public class ShopItemManager {
    public ItemStack getSword() {
        // 坏味道：一旦想修改名字或材质，必须修改 Java 代码并重新编译
        var item = new ItemStack(Material.DIAMOND_SWORD);
        item.editMeta(meta -> {
            meta.displayName(MiniMessage.miniMessage().deserialize("<red>炎龙之剑"));
            meta.setCustomModelData(1001);
        });
        return item;
    }
}
```

## 2. 正确示范 (Config Injection)

Feature 模块应该包含一个负责加载配置的类（如 `ShopConfig` 或 `ShopItemLoader`）。

### 2.1 配置文件结构 (`resources/items.yml`)

建议使用 Feature 名称作为一级 Key，实现逻辑上的命名空间隔离。

```yaml
features:
  shop:
    fire-sword:
      material: DIAMOND_SWORD
      name: "<red>炎龙之剑"
      custom-model-data: 1001
      lore:
        - "<gray>攻击力: <red>+50"
  fishing:
    golden-rod:
      material: FISHING_ROD
      name: "<gold>黄金钓竿"
```

### 2.2 Java 加载逻辑 (`features/shop/ShopConfig.java`)

```java
package com.example.plugin.features.shop;

import org.bukkit.configuration.ConfigurationSection;
import org.bukkit.inventory.ItemStack;
import net.kyori.adventure.text.minimessage.MiniMessage;
import com.example.plugin.utils.ItemBuilder; // 假设有一个通用的构建器

public class ShopConfig {
    
    private final Plugin plugin;
    private ItemStack fireSword;

    public ShopConfig(Plugin plugin) {
        this.plugin = plugin;
    }

    public void reload() {
        // 定位到 features.shop 节点
        ConfigurationSection section = plugin.getConfig().getConfigurationSection("features.shop.fire-sword");
        
        if (section == null) {
            plugin.getLogger().warning("Missing config: features.shop.fire-sword");
            return;
        }

        // 从配置构建物品
        this.fireSword = ItemBuilder.fromConfig(section).build();
    }

    // Manager 调用此方法获取缓存的物品
    public ItemStack getFireSword() {
        return this.fireSword.clone(); // 返回克隆以防止修改源
    }
}
```

### 2.3 在 Feature 中组装

```java
public class ShopFeature implements GameFeature {
    private final ShopConfig config;
    private final ShopManager manager;

    public void onEnable() {
        this.config = new ShopConfig(plugin);
        this.config.reload(); // 初始加载
        
        // 将配置注入给 Manager
        this.manager = new ShopManager(config); 
    }
}
```

## 3. 优势

1.  **模块化**: `ShopConfig` 只关心商店的配置，不关心其他。
2.  **热重载**: 只需要调用 `ShopConfig.reload()` 即可更新物品，无需重启服务器。
3.  **可维护性**: 策划人员只需要修改 `items.yml`，无需接触 Java 代码。
