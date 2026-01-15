# 事件驱动优化 (Event-Driven Optimization)

## 核心原则

**"Don't call us, we'll call you."**
尽量避免使用 `runTaskTimer` 进行高频（1 tick - 20 ticks）轮询检查，特别是当检查逻辑涉及 IO、复杂计算或流式操作时。

## 场景分析

### 场景：烹饪锅热源检查
- **需求**: 只有当锅下方有火时，烹饪才进行。
- **低效做法 (Polling)**:
  ```java
  // 每 tick 运行
  public void tick() {
      if (world.getBlockAt(down).getType() == Material.CAMPFIRE) { // 每次都读取 World
          cook();
      }
  }
  ```
- **问题**: 99% 的时间热源状态没有改变，但每 tick 都在消耗 CPU 进行区块访问和类型检查。

### 优化做法 (Event-Driven + Caching)

1.  **状态缓存**: 在对象内部维护一个 `cachedHeated` 布尔值。
2.  **事件触发**: 仅在可能改变状态的事件发生时更新缓存。
    - 玩家放置/破坏方块。
    - 营火熄灭/点燃。
    - **折中方案 (Lazy Check)**: 如果监听所有事件太复杂，可以在**交互时**（如打开 GUI、放入物品）更新一次缓存，并在 tick 中直接信任缓存。

```java
// ✅ 优化做法
public class CookingPot {
    private boolean isHeated = false;
    private long lastCheckTime = 0;

    // 在 tick 中只读内存变量
    public void tick() {
        if (!isHeated) return;
        // ... cooking logic
    }

    // 仅在关键交互或低频间隔更新状态
    public void updateHeatState() {
        this.isHeated = checkRealBlockState();
    }
}
```

## 收益
- **CPU**: 几乎降至 0 开销。
- **扩展性**: 即使有 10,000 个锅，只要它们不被同时交互，服务器负载就不会线性增长。
