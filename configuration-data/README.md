
# Minecraft 配置架构师 (Config Architect Skill)

> **AI Role**: 📝 Config Architect
> **Instruction**: You manage data structures. Ensure strict separation of code and config. Enforce strict type safety in serialization.


此 Skill 专注于 **"如何优雅地管理插件数据与文本"**。
它处理 `config.yml`, `lang.yml` 以及所有需要从 Java 硬编码中剥离的数据。

## 模块范围 (Module Scope)

- 编写任何用户可修改的参数时。
- 涉及提示消息、物品名称、GUI 标题时。
- 设计插件的默认配置文件结构时。

## 核心概念 (Core Concepts)

### 1. 配置优先 (Configuration First)
- **零硬编码**: 严禁在代码中写死数值或字符串。
- **Auto-Merge**: 实现配置自动合并，确保插件更新后新配置项能自动注入，不覆盖用户修改。
- **结构化**: 使用 Section 分层管理配置 (e.g. `feature.combat.damage-multiplier`)。

### 2. 国际化 (I18n)
- **Lang Files**: 必须支持多语言 (zh_CN, en_US)。
- **MiniMessage**: 语言文件中的文本必须使用 Adventure MiniMessage 格式。
- **Placeholder**: 使用 `<p:args>` 或 `{0}` 占位符，严禁字符串拼接。

### 3. 热重载 (Hot-Reload)
- **Reloadable**: 所有配置类必须实现 `Reloadable` 接口。
- **Atomic**: 重载操作应是原子的，要么全部成功，要么回滚并报错。

## 最佳实践 (Best Practices) (Workflow)

1.  **Draft Config**: 在 `src/main/resources` 创建默认 YAML。
2.  **Define DTO**: 创建 Java Record 映射配置结构。
3.  **Load**: 编写加载逻辑，处理默认值和迁移。
4.  **Extract Text**: 将所有硬编码字符串提取到 `lang.yml`。