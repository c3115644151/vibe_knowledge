# Configuration Over Hardcoding (配置优于硬编码)

**原则**: 所有的游戏逻辑参数、物品ID、GUI布局、提示文本等“易变数据”必须从配置文件加载，严禁硬编码在 Java 代码中。

## 1. 为什么禁止硬编码？
- **灵活性**: 服务器管理员无需重新编译插件即可修改游戏机制。
- **兼容性**: 硬编码的 ID (如 `farmersdelight:flint_knife`) 可能在不同 Mod/插件版本中发生变化。
- **可维护性**: 将数据与逻辑分离，使代码更清晰。

## 2. 什么是“易变数据”？
以下内容绝对禁止硬编码：
- **物品 ID / Material**: 如 `DIAMOND_SWORD`, `farmersdelight:knife`。
- **数值参数**: 如 冷却时间、伤害值、堆叠数量、概率。
- **GUI 布局**: 标题、图标位置、填充物。
- **提示文本**: 聊天消息、Lore、Title (应使用 `messages.yml` 或 `I18n`)。

## 3. 正确示范

### ❌ 错误 (Hardcoding)
```java
public class KnifeValidator {
    // 错误：将物品 ID 硬编码在 Set 中
    private static final Set<String> ALLOWED_KNIVES = Set.of(
        "farmersdelight:flint_knife",
        "farmersdelight:iron_knife"
    );

    public boolean isValid(ItemStack item) {
        return ALLOWED_KNIVES.contains(getId(item));
    }
}
```

### ✅ 正确 (Configuration)
**config.yml**:
```yaml
allowed_knives:
  - farmersdelight:flint_knife
  - farmersdelight:iron_knife
```

**Java**:
```java
public class KnifeValidator {
    private final Set<String> allowedKnives = new HashSet<>();
    private final JavaPlugin plugin;

    public KnifeValidator(JavaPlugin plugin) {
        this.plugin = plugin;
        loadConfig();
    }

    public void loadConfig() {
        allowedKnives.clear();
        // 提供默认值以防配置缺失
        List<String> list = plugin.getConfig().getStringList("allowed_knives");
        if (list.isEmpty()) {
            allowedKnives.add("farmersdelight:flint_knife");
        } else {
            allowedKnives.addAll(list);
        }
    }

    public boolean isValid(ItemStack item) {
        return allowedKnives.contains(getId(item));
    }
}
```

## 4. 默认值策略
读取配置时，**必须**考虑到配置缺失的情况。
- 使用 `config.getString("path", "defaultValue")`。
- 或者在加载后检查集合是否为空，如果为空则填入默认的“硬编码”值（仅作为 Fallback）。
