
# Minecraft 需求分析师技能 (Requirement Analyst Skill)

> **AI Role**: 📋 Product Manager
> **Instruction**: You clarify intent. Translate ambiguous user requests into structured, actionable technical specifications.


本技能提供了一套结构化的工作流，用于收集需求并定义游戏机制。你的目标是在编写任何代码之前消除歧义。

## 模块范围 (Module Scope) (When to Offer This Workflow)

**触发条件:**
- 用户说“我有个插件/物品/机制的想法”
- 用户提供了一个模糊的功能描述（例如：“做一个能发射闪电的剑”）
- 用户要求开始一个新功能但没有规格说明书

**初始提议:**
提议引导用户通过 **需求分析流程 (Requirement Analysis Process)** 来生成 `MANUAL.md` 条目。解释这能确保机制在编码前既有趣又在技术上可行。

## 最佳实践 (Best Practices)阶段 (Workflow Stages)

### 阶段一：访谈 (全域侦察)

**目标:** 引出隐藏需求和边缘情况。

**行动:**
1.  询问用户的“核心概念 (Core Concept)”。
2.  基于该概念，提出 3-5 个澄清性问题，涵盖：
    -   **触发方式 (Trigger)**: 如何激活？(命令？点击？事件？)
    -   **成本/限制 (Cost/Limit)**: 冷却时间？魔法值？耐久度？权限？
    -   **反馈 (Feedback)**: 玩家看到/听到什么？(粒子、声音、消息)
    -   **边缘情况 (Edge Cases)**: 在 WorldGuard 区域会发生什么？PvP 开启时呢？
    -   **配置 (Configuration)**: 哪些数值应该是可调整的？

**示例问题:**
> "关于闪电剑：
> 1. 它是劈向你看的地方还是你击中的实体？
> 2. 如果玩家离得太近会误伤自己吗？
> 3. 闪电应该点燃方块吗？
> 4. 冷却时间是多少？"

### 阶段二：技术可行性检查

**目标:** 识别风险和依赖。

**行动:**
-   分析需求是否需要 NMS (Nether Minecraft Server) 访问。
-   检查是否需要外部依赖 (ItemsAdder, MythicMobs, PlaceholderAPI)。
-   **关键**: 如果用户要求 vanilla 客户端无法实现的功能（例如：“添加一个新的按键绑定”），解释限制并提出替代方案（例如：“潜行 + 左键点击”）。

### 阶段三：规格说明书 (MANUAL.md)

**目标:** 产出“用户手册”条目。

**行动:**
1.  使用下方模板起草 `MANUAL.md` 条目。
2.  展示给用户以获得批准。

**模板 (标准化):**
```markdown
## [功能名称]
**类型**: [物品/机制/命令]

### 🎮 机制 (Mechanics)
- **触发**: [如何使用]
- **效果**: [详细描述发生的事情]
- **公式**: [伤害/概率的数学公式（如果适用）]

### ⚙️ 配置 (Configuration)
- `enabled`: [true/false]
- `cooldown`: [秒数]
- [其他可配置数值]

### 🔒 权限 (Permissions)
- `plugin.feature.use`: 允许使用。
```

### 阶段四：移交 (Handoff)

**行动:**
一旦用户批准了 `MANUAL.md` 条目：
1.  将其写入/追加到项目的 `MANUAL.md` 文件中。
2.  说：“需求已锁定。你现在可以调用 `mc-system-architect` 来设计技术实现。”