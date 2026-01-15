# 可选依赖隔离模式 (Optional Dependency Isolation)

## ⚠️ 问题背景 (The Pitfall)
在 Bukkit/Spigot 插件开发中，我们经常需要支持“软依赖” (Soft Dependency)，例如：如果不安装 ItemsAdder，插件也能运行，但安装了会有额外功能。

一个常见的**错误做法**是：在核心逻辑类（如 `StoveListener`）中直接导入并使用软依赖的类（如 `CustomBlockPlaceEvent`）。

### 错误示例 (Anti-Pattern)
```java
// ❌ 错误：在核心监听器中直接 Import 可选依赖的类
import dev.lone.itemsadder.api.Events.CustomBlockPlaceEvent; 

public class StoveListener implements Listener {
    
    // 这个方法会导致整个类加载失败，或者在 registerEvents 时报错
    @EventHandler
    public void onIAPlace(CustomBlockPlaceEvent event) { 
        // ... 逻辑
    }
    
    @EventHandler
    public void onNormalInteract(PlayerInteractEvent event) {
        // ... 正常逻辑
    }
}
```

### 崩溃原理 (Root Cause)
1.  **类加载验证 (Class Verification)**: 当 JVM 加载 `StoveListener` 类时，或者 Bukkit 尝试反射扫描 `@EventHandler` 方法时，它必须解析所有参数类型。
2.  **缺失类异常**: 如果服务器上没有安装 ItemsAdder，`CustomBlockPlaceEvent` 类就不存在。
3.  **全盘崩溃**: 此时，JVM 抛出 `NoClassDefFoundError` 或 `ClassNotFoundException`。这不仅会导致那个特定的 EventHandler 失效，还会导致**整个 Listener 类注册失败**。
4.  **后果**: 即使是与 ItemsAdder 无关的 `onNormalInteract` 也就此失效，导致核心功能瘫痪。

---

## ✅ 解决方案 (Best Practice)

采用 **监听器分离 (Listener Segregation)** 策略。将所有依赖于特定插件的事件处理逻辑，剥离到单独的类中。

### 1. 独立监听器类
创建一个专门处理该依赖的监听器类。

```java
// ✅ 正确：独立的监听器类
public class ItemsAdderListener implements Listener {
    private final MyPlugin plugin;

    public ItemsAdderListener(MyPlugin plugin) {
        this.plugin = plugin;
    }

    @EventHandler
    public void onCustomBlockPlace(CustomBlockPlaceEvent event) {
        // ... 安全地使用 IA API
    }
}
```

### 2. 条件注册 (Conditional Registration)
在主类 `onEnable` 中，检查插件是否存在，再决定是否注册该监听器。

```java
@Override
public void onEnable() {
    // 注册核心监听器 (无外部依赖)
    getServer().getPluginManager().registerEvents(new StoveListener(this), this);

    // 条件注册可选监听器
    if (getServer().getPluginManager().isPluginEnabled("ItemsAdder")) {
        getLogger().info("ItemsAdder detected! Enabling IA integration...");
        getServer().getPluginManager().registerEvents(new ItemsAdderListener(this), this);
    }
}
```

## 🧠 核心原则 (Core Principles)
1.  **保持核心纯净**: 核心业务逻辑（Core Domain）不应直接 `import` 任何不确定是否存在的外部类。
2.  **依赖倒置/隔离**: 将外部集成逻辑推向边缘（Edge），封装在专门的 Adapter 或 Listener 中。
3.  **防御性编程**: 即使在方法内部使用 `Class.forName` 反射调用有时可行，但对于**事件监听器**（Event Listeners），必须物理隔离类文件，因为 Bukkit 的注册机制会扫描所有方法签名。
