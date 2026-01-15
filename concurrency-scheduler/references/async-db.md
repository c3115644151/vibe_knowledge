# 示例：异步数据库加载
展示如何安全地在异步线程加载数据，并在同步线程更新 UI。

```java
public void loadPlayerData(UUID uuid) {
    // 1. 异步读取 (IO Bound)
    CompletableFuture.supplyAsync(() -> {
        try (Connection conn = dataSource.getConnection()) {
            PreparedStatement ps = conn.prepareStatement("SELECT * FROM stats WHERE uuid = ?");
            ps.setString(1, uuid.toString());
            return ps.executeQuery();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }).thenAccept(rs -> {
        // 2. 回到主线程更新 Bukkit (CPU Bound / API)
        Bukkit.getScheduler().runTask(plugin, () -> {
            try {
                if (rs.next()) {
                    // 安全调用 Bukkit API
                    Player player = Bukkit.getPlayer(uuid);
                    if (player != null) {
                        player.sendMessage("数据加载完毕！");
                    }
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        });
    }).exceptionally(ex -> {
        plugin.getLogger().severe("加载失败: " + ex.getMessage());
        return null;
    });
}
```
