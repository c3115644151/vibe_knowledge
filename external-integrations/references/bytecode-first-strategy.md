# Bytecode First Strategy (字节码优先策略)

## Context
当集成闭源插件、文档缺失或使用了 Shading/Relocation 技术的第三方库时，文档和经验往往是不可靠的。

## The Problem
- **Class Not Found**: 依赖库被重定向 (Relocated) 到新的包名下 (e.g., `okio` -> `plugin.libs.okio`)。
- **Method Mismatch**: API 版本更新导致方法签名变更，反射调用抛出 `NoSuchMethodException`。
- **Hidden API**: 某些关键功能仅作为内部 API 存在，文档未公开。

## The Solution
在编写任何集成代码 (Hook) 之前，**必须**直接检查运行时环境中的字节码。

### 1. Jar Scanning (Jar 扫描)
使用 `jar tf` 命令列出 Jar 包内的所有文件，快速定位目标类的真实路径。

```bash
# 查找 CraftEngine 中关于 NBT 的类
jar tf plugins/CraftEngine.jar | grep "NBT"
# 输出: net/momirealms/craftengine/libraries/nbt/CompoundTag.class
```

### 2. Method Signature Verification (签名验证)
使用 `javap` 命令反编译目标类，获取准确的方法签名（包括参数类型和返回值）。

```bash
# 查看 place 方法的具体签名
javap -cp plugins/CraftEngine.jar -p net.momirealms.craftengine.bukkit.api.CraftEngineBlocks
```

### 3. Dynamic Reflection (动态反射)
根据扫描结果，编写能够适应不同签名的防御性反射代码。

```java
// 错误示范：硬编码类名
Class<?> clazz = Class.forName("net.momirealms.sparrow.nbt.CompoundTag");

// 正确示范：动态探测与回退
Class<?> clazz;
try {
    // 优先尝试 Jar 包中发现的 Shaded 路径
    clazz = Class.forName("net.momirealms.craftengine.libraries.nbt.CompoundTag");
} catch (ClassNotFoundException e) {
    // 回退到原始路径
    clazz = Class.forName("net.momirealms.sparrow.nbt.CompoundTag");
}
```

## Checklist
- [ ] 目标插件 Jar 是否存在于工作区？
- [ ] 是否已运行 `jar tf` 确认核心类的完整包名？
- [ ] 是否已运行 `javap` 确认方法的参数类型（特别是重载方法）？
- [ ] 反射代码是否处理了 `ClassNotFoundException` 和 `NoSuchMethodException`？
