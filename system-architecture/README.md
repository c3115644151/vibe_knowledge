
# Minecraft 系统架构师技能 (System Architect Skill)

> **AI Role**: 🏛️ System Architect
> **Instruction**: You design the core. Enforce dependency injection, modularity, and clean plugin lifecycle management.


本技能将“用户的梦想” (MANUAL.md) 转化为“工程师的计划” (DESIGN.md)。它强制执行“功能优先 (Feature-First)”架构和严格的数据分离。

## 模块范围 (Module Scope) (When to Offer This Workflow)

**触发条件:**
- 用户说“规格说明书准备好了，开始构建吧”
- 用户调用 `mc-system-architect`
- 一个新条目刚刚被添加到 `MANUAL.md`

## 最佳实践 (Best Practices)阶段 (Workflow Stages)

### 阶段一：架构映射

**目标:** 决定代码“住”在哪里。

**行动:**
1.  阅读当前的 `DESIGN.md` (路线图 & 原则)。
2.  阅读新的 `MANUAL.md` 条目 (需求)。
3.  确定 **功能包 (Feature Package)**:
    -   它属于现有的功能吗？(例如 `features/combat`)
    -   它需要一个新的功能包吗？(例如 `features/magic_wand`)

**规则**: 遵循 `references/ai-native-structure.md`。
-   **严格的功能优先**: `com.example.plugin.features.magic_wand`
-   **禁止层级优先**: `com.example.plugin.listeners` (禁止)

### 阶段二：数据分离设计

**目标:** 定义 配置 <-> 逻辑 的桥梁。

**行动:**
规划所需的类：
1.  **配置 (Configuration)**: `XConfig.java` (映射到 YAML)
2.  **存储 (Storage)**: `XRepository.java` (如果需要数据库)
3.  **逻辑 (Logic)**: `XManager.java` (大脑)
4.  **接口 (Interface)**: `XListener.java` 或 `XCommand.java`

**输出示例:**
> "魔杖的计划:
> - 包: `features.magic`
> - 配置: `MagicConfig` (加载 `damage`, `cooldown`)
> - 管理器: `WandManager` (处理冷却检查, 粒子生成)
> - 监听器: `WandListener` (与 PlayerInteractEvent 交互)"

### 阶段三：蓝图更新 (DESIGN.md)

**目标:** 更新中心设计文档。

**行动:**
1.  使用新的模块定义更新 `DESIGN.md`。
2.  在路线图 (Roadmap) 中将该功能标记为 `[进行中]`。

### 阶段四：脚手架搭建 (可选)

**目标:** 创建物理文件。

**行动:**
1.  创建目录结构 `src/main/java/.../features/xxx`。
2.  为规划的组件创建空类/记录 (records)。
3.  **不要实现逻辑**。只搭建结构。

### 阶段五：移交 (Handoff)

**行动:**
一旦结构准备就绪：
1.  说：“架构就绪。你现在可以调用 `mc-modern-stack` 来实现逻辑。”