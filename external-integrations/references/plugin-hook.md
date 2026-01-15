# 示例：安全的插件 Hook 写法

## 1. 插件描述文件 (plugin.yml)
```yaml
name: PastureSong
main: com.example.pasturesong.PastureSong
version: 1.0.0
# 软依赖：对方不装我也能跑，装了我有更多功能
softdepend: [CuisineFarming]
```

## 2. Hook 管理器 (CuisineHook.java)
```java
public class CuisineHook {
    private final boolean enabled;
    private final Plugin targetPlugin;

    public CuisineHook() {
        // 安全检查：插件是否存在且已启用
        Plugin plugin = Bukkit.getPluginManager().getPlugin("CuisineFarming");
        this.enabled = plugin != null && plugin.isEnabled();
        this.targetPlugin = plugin;
        
        if (enabled) {
            Bukkit.getLogger().info("[PastureSong] 成功连接 CuisineFarming!");
        }
    }

    public boolean isEnabled() {
        return enabled;
    }

    /**
     * 安全地获取饲料数据
     * 如果插件未安装，返回默认值或空
     */
    public Optional<Double> getFodderNutrition(ItemStack item) {
        if (!enabled) return Optional.empty();
        
        // 只有在 enabled 为 true 时才访问对方的类
        // 这样可以避免 NoClassDefFoundError
        try {
            return getNutritionInternal(item);
        } catch (Exception e) {
            e.printStackTrace();
            return Optional.empty();
        }
    }

    // 将具体的 API 调用隔离在独立方法中
    private Optional<Double> getNutritionInternal(ItemStack item) {
        // 假设 CuisineFarming 提供了这个 API
        // com.example.cuisinefarming.api.CuisineAPI
        return com.example.cuisinefarming.api.CuisineAPI.getNutrition(item);
    }
}
```

## 3. 调用处
```java
// 在业务逻辑中
public void onCowEat(Cow cow, ItemStack food) {
    if (hooks.getCuisineHook().isEnabled()) {
        hooks.getCuisineHook().getFodderNutrition(food).ifPresent(nutrition -> {
            cow.heal(nutrition);
            player.sendMessage("牛吃到了美味的饲料！");
        });
    } else {
        // 降级处理：普通小麦逻辑
        if (food.getType() == Material.WHEAT) {
            cow.heal(1.0);
        }
    }
}
```
