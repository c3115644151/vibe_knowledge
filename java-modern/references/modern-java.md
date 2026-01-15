# 现代 Java 规范 (Modern Java)

## 1. Record (记录类)
优先使用 `record` 代替不可变的 POJO/DTO。

```java
// 定义
public record CropData(String id, double growthRate, List<String> drops) {}

// 使用
var crop = new CropData("wheat", 1.0, List.of("WHEAT", "SEEDS"));
System.out.println(crop.id());
```

## 2. Pattern Matching (模式匹配)
使用 `instanceof` 的模式匹配功能，减少强制转换。

```java
// 旧写法
if (sender instanceof Player) {
    Player player = (Player) sender;
    // ...
}

// ✅ 新写法
if (sender instanceof Player player) {
    player.sendMessage("Hello!");
}
```

## 3. Switch Expressions (Switch 表达式)
使用箭头语法，更简洁且无穿透风险。

```java
var xpReward = switch(type) {
    case WHEAT -> 10;
    case CARROT, POTATO -> 20; // 合并 case
    case MELON -> {
        System.out.println("Big reward!");
        yield 50;
    }
    default -> 0;
};
```
