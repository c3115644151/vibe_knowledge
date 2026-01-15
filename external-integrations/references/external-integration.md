# 示例：集成 RSSeasons 季节插件

## 1. 错误的做法 (The Wrong Way)
AI 基于猜测编写代码，导致编译错误或运行时报错。

```java
// ❌ 错误：猜测的 API
Season season = RSSeasons.getSeason(); // 方法不存在！
```

## 2. 正确的做法 (The Detective Way)

### 步骤 1: 寻找真相
使用工具搜索仓库和 API 文档：
`WebSearch(query="RSSeasons minecraft github API docs")`
-> 找到 Wiki 或 JavaDocs 页面。

### 步骤 2: 验证信息
确认 API 的调用方式。
*搜索结果*: "Use `SeasonSystem.getInstance().getSeason(world)` to get current season."

### 步骤 3: 编写代码
基于确切信息编写 Hook。

```java
public class SeasonHook {
    private boolean enabled = false;

    public SeasonHook() {
        if (Bukkit.getPluginManager().isPluginEnabled("RSSeasons")) {
            enabled = true;
        }
    }

    public String getCurrentSeason(World world) {
        if (!enabled) return "UNKNOWN";
        try {
            // ✅ 正确：基于查询结果的 API
            return SeasonSystem.getInstance().getSeason(world).getName();
        } catch (Throwable e) {
            e.printStackTrace();
            return "ERROR";
        }
    }
}
```
