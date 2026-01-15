
# Minecraft Java 语言大师 (Java Virtuoso Skill)

> **AI Role**: ☕ Java Virtuoso
> **Instruction**: You enforce code quality. MANDATORY: Use Java 21+ features (Records, Switch Expressions) and modern streams.


此 Skill 定义了 **"如何编写高质量的 Java 代码"**。
它是所有其他技能的基础，确保代码简洁、安全、可维护。

## 模块范围 (Module Scope)

- 编写任何 `.java` 文件时。
- 定义新的数据结构（DTO, POJO）时。
- 处理可能为 null 的输入时。
- 涉及时间参数（秒、Tick）转换时。

## 核心概念 (Core Concepts)

### 1. 现代 Java 规范 (Modern Java)
- **Record**: 数据类必须用 `record`，禁止 Lombok `@Data`。
- **Switch**: 使用 Switch Expression (箭头语法) 和 Pattern Matching。
- **Var**: 局部变量类型推断。
- **Text Block**: 多行字符串。

### 2. 防御性编程 (Defensive Coding)
- **Null Safety**: 严禁信任 `getItemMeta()` 等可能返回 null 的 API。
- **Input Validation**: 插件命令参数必须校验。
- **Fail Fast**: 尽早抛出异常，而不是让错误蔓延。

### 3. 时间单位纪律 (Time Unit Discipline)
- **命名**: 变量名必须带后缀 `Ticks`, `Seconds`, `Millis`。
- **配置**: 配置文件默认单位明确化。
- **转换**: 避免手动 `* 20`，使用工具类或清晰的常量。

### 4. 可观测性 (Observability)
- **Debug 开关**: 必须通过 `config.debug` 控制日志输出。
- **日志分级**: 区分 Info, Warn, Error 和 Debug。

## 最佳实践 (Best Practices) (Workflow)

1.  **Draft**: 编写代码初稿。
2.  **Modernize**: 检查是否使用了旧式语法（如 `if-else` 链代替 `switch`，手动 Getter/Setter 代替 `record`）。
3.  **Defend**: 检查所有外部输入（API, Config, Command）的 Null 安全性。
4.  **Unit Check**: 检查所有 `int`, `long` 类型的时间变量是否带有单位后缀。