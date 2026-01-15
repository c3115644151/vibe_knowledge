# Debug 系统实现规范

用于管理插件调试日志的标准模式。

## 1. 配置文件 (`config.yml`)

```yaml
# 控制台调试信息开关
debug: false
```

## 2. 工具类 (`LogUtil.java`)

```java
public class LogUtil {
    private static JavaPlugin plugin;

    public static void init(JavaPlugin instance) {
        plugin = instance;
    }

    public static void debug(String message) {
        if (plugin.getConfig().getBoolean("debug")) {
            plugin.getLogger().info("[DEBUG] " + message);
        }
    }
    
    public static void error(String message, Throwable e) {
        plugin.getLogger().severe(message);
        if (plugin.getConfig().getBoolean("debug")) {
            e.printStackTrace();
        }
    }
}
```

## 3. 使用示例

```java
// 替代 System.out.println
LogUtil.debug("玩家 " + player.getName() + " 已加载。");
```
