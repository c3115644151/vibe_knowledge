
# Minecraft 构建工程师 (Build Engineer Skill)

> **AI Role**: 🛠️ Build Engineer
> **Instruction**: You govern the build process. Enforce Gradle Kotlin DSL, ShadowJar best practices, and dependency management.


此 Skill 专注于 **"如何将代码打包成可运行的插件"**。
它处理构建工具 (Maven/Gradle)、依赖管理和 Jar 包结构。

## 模块范围 (Module Scope)

- 配置 `pom.xml` 或 `build.gradle` 时。
- 遇到 `NoClassDefFoundError` (依赖缺失) 或 `NoSuchMethodError` (版本冲突) 时。
- 需要将第三方库 (如 HikariCP, Kotlin) 打包进插件时。
- 设计多模块项目 (Core + NMS Modules) 时。

## 核心概念 (Core Concepts)

### 1. 依赖打包 (Shadowing/Shading)
- **Shade**: 使用 Maven Shade Plugin 将依赖打入 Jar。
- **Relocation**: 必须重定向包名 (e.g. `com.zaxxer` -> `me.plugin.libs.zaxxer`)，防止与其他插件冲突。
- **Minimize**: 剔除未使用的类以减小体积。

### 2. 多模块架构 (Multi-Module)
- **Core**: 包含业务逻辑，不依赖 NMS。
- **Adapter**: 定义 NMS 接口。
- **v1_21_R1**: 实现具体版本的 NMS 代码。
- **Dist**: 负责将所有模块打包成最终插件。

### 3. 构建优化 (Build Optimization)
- **Resources**: 正确处理 `plugin.yml` 中的变量替换 (`${project.version}`)。
- **Profile**: 使用 Maven Profile 区分开发环境和生产环境。

## 最佳实践 (Best Practices) (Workflow)

1.  **Define Dep**: 在 `pom.xml` 添加依赖。
2.  **Config Shade**: 如果是 `compile` 范围依赖，配置 Shade Relocation。
3.  **Build**: 执行 `mvn clean package`。
4.  **Verify**: 检查生成的 Jar 内容，确保没有意外引入不必要的库。