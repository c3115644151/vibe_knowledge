# 配置劫持模式 (Config Hijack Pattern)

## 🎯 核心概念

**配置劫持 (Config Hijack)** 是一种在目标插件 API 缺失、文档过时或存在严重缺陷（如 AI 幻觉导致的虚假 API）时的**终极集成手段**。

其核心思想是：**完全绕过插件提供的 Java API，直接读取其磁盘上的配置文件（YAML/JSON），在内存中重建所需的数据映射。**

## ⚠️ 反模式警示 (Anti-Pattern Alert)

### 💀 API 幻觉陷阱 (The API Hallucination Trap)
在集成闭源或文档稀缺的插件（如 CraftEngine, ItemsAdder）时，LLM (包括 DeepWiki) 极易产生幻觉，编造出语义合理但不存在的 API。

*   **典型症状**:
    *   AI 坚称存在 `getCustomItemName(id)` 或 `getItemStack(id)`。
    *   IDE 报错 `Cannot resolve method`。
    *   反编译 jar 包后发现根本没有对应的方法，或者方法签名完全不同。
    *   **无效循环**: 开发者反复询问 AI -> AI 换一个不存在的方法名 -> 开发者继续尝试 -> 浪费数小时。

*   **熔断机制**:
    *   当连续 **2次** 尝试 AI 推荐的 API 失败（编译错误或运行时无效果）时，**立即停止**询问 API 相关问题。
    *   **转而分析**:
        1.  插件的数据存在哪？（通常是 YAML/JSON 配置文件）
        2.  我能否直接读取这些文件？

## 🛠️ 最佳实践 (Best Practices)

### 1. 目录定位与插件发现
不要假设插件文件夹名就是插件名。用户可能重命名文件夹，或者插件名与文件夹名不一致（如 `CraftEngine` vs `CraftEngine-Reborn`）。

```java
// 推荐：双重检查策略
Plugin plugin = Bukkit.getPluginManager().getPlugin("TargetPlugin");
if (plugin == null) {
    // 降级：扫描所有插件的数据文件夹名
    for (Plugin p : Bukkit.getPluginManager().getPlugins()) {
        if (p.getDataFolder() != null && p.getDataFolder().getName().equals("TargetPlugin")) {
            plugin = p;
            break;
        }
    }
}
```

### 2. 递归扫描与容错解析
配置文件通常嵌套在多层目录中。必须使用递归扫描，并对每个文件进行 `try-catch` 保护，防止单个文件格式错误导致整个加载失败。

```java
// 范式：递归加载 + 结构嗅探
private void loadRecursively(File dir) {
    for (File file : dir.listFiles()) {
        if (file.isDirectory()) loadRecursively(file);
        else if (file.getName().endsWith(".yml")) loadFile(file);
    }
}

private void loadFile(File file) {
    YamlConfiguration config = YamlConfiguration.loadConfiguration(file);
    // 策略 A: 直接检查 Key (Item本身就是Root)
    // 策略 B: 检查嵌套 Section (Item在 items: 或 templates: 下)
    // 必须同时支持多种结构，因为插件作者可能在不同版本改变结构
}
```

### 3. 模糊匹配 (Fuzzy Matching)
配置文件中的 Key 往往不带命名空间（如 `cabbage`），而代码中引用的 ID 往往带命名空间（如 `farmersdelight:cabbage`）。

*   **建立映射时**: 存储完整 Key。
*   **读取时**: 如果完整 Key 匹配失败，尝试剥离命名空间后再次匹配。

```java
public String getName(String fullId) {
    // 1. 精确匹配
    if (cache.containsKey(fullId)) return cache.get(fullId);
    
    // 2. 模糊匹配 (去除 namespace)
    if (fullId.contains(":")) {
        String idOnly = fullId.split(":")[1];
        if (cache.containsKey(idOnly)) return cache.get(idOnly);
    }
    return null;
}
```

### 4. 性能权衡
*   **时机**: 在 `onEnable` 或 `PostWorldInitializeEvent` 中异步加载。
*   **缓存**: 必须将解析结果缓存到 `static Map` 中，严禁在运行时实时读取 IO。
*   **日志**: 打印加载统计（如“已加载 500 个物品名称”），但只在开发模式或出错时打印详细内容。

## 案例参考
*   **场景**: 获取 CraftEngine 自定义物品的本地化名称。
*   **问题**: API 无法返回正确的 DisplayName，总是返回原版材质名（如“牛排”）。
*   **解决**: 直接读取 `plugins/CraftEngine/resources/**/*.yml`，提取 `data.item-name` 字段，建立 `id -> item-name` 映射。
