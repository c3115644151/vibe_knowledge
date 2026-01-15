# 时间单位纪律 (Time Unit Discipline)

## 背景 (Context)
在 Minecraft 开发中，时间单位极其容易混淆：
- **Ticks**: 游戏逻辑的基本单位 (1秒 = 20 Ticks)。
- **Seconds/Minutes**: 配置文件或人类可读的常用单位。
- **Millis**: Java 线程或 `System.currentTimeMillis()` 使用的单位。

**典型灾难**: 误以为配置文件的 `600` 是秒，或者误以为代码里的 `delay` 需要手动乘以 20，导致时间被错误放大 20 倍（如 10分钟变成了 3.3小时，或 30秒变成 10分钟）。

## 核心法则 (Core Rules)

### 1. 命名即文档 (Naming as Documentation)
所有涉及时间的变量、字段、参数，**必须**在命名中包含单位后缀。
- ❌ `int time;`
- ❌ `int duration;`
- ✅ `int timeTicks;`
- ✅ `int durationSeconds;`
- ✅ `long lastCookTimeMillis;`

### 2. 配置默认为 Ticks (Config Defaults to Ticks)
除非另有明确说明（如 `auto_save_interval_seconds`），否则所有 Minecraft 机制相关的配置项（如冷却、持续时间、延迟）均应被视为 **Ticks**。
- **读取时**: `int cookTimeTicks = config.getInt("cook_time");`
- **严禁**: `int cookTime = config.getInt("cook_time") * 20; // 盲目乘20是万恶之源`

### 3. 显式转换 (Explicit Conversion)
如果必须进行单位转换，必须显式写出，最好使用工具方法或常量，并在变量名上体现变化。

```java
// Good
int durationSeconds = config.getInt("duration_seconds");
int durationTicks = durationSeconds * 20;

// Good
int cooldownTicks = config.getInt("cooldown"); // Assume ticks by default

// Bad
int time = config.getInt("time");
schedule(time * 20); // Is time seconds? Are you sure?
```

## 案例分析 (Case Study)

### 错误案例 (The Anti-Pattern)
```java
// config.yml
// cooking_time: 600 (Intended as 30 seconds / 600 ticks)

// Java
int time = config.getInt("cooking_time");
// ... later ...
item.setCooldown(time * 20); // Result: 12000 ticks (10 minutes!)
```

### 正确案例 (The Pattern)
```java
// Java
int cookingTimeTicks = config.getInt("cooking_time");
item.setCooldown(cookingTimeTicks);
```
