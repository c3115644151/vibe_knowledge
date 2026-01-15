# Vibe Coding & Agent Skills: The Manifesto

> **定位**: 本文档是系统的“元知识库”。当需要深入理解 Skills 设计原理、维护文档规范或进行高阶自我进化时，请查阅此文档。

## 🏛️ 第一章：核心哲学 (The Core Philosophy)

### 1.1 Vibe Coding 是什么？
Vibe Coding 不仅仅是“写代码”，而是一种 **Planning-Driven (规划驱动)** + **Context-Aware (上下文感知)** + **Synchronous Human-in-the-Loop (同步人机回环)** 的全新交互范式。
- **传统模式**: 用户给指令 -> AI 写代码 -> 报错 -> 用户修 -> AI 修 (无尽的 Loop)。
- **Vibe 模式**: 
    1.  **Context Fixation**: 先把“想法”变成“文档” (Memory Bank)，锚定上下文。
    2.  **Skill Routing**: 根据任务类型，挂载最专业的技能模块。
    3.  **Execution**: 基于稳固的上下文和专业的技能执行。
    4.  **Interactive Feedback**: 在关键节点（构建后、迷茫时）主动挂起，等待用户实时确认，确保不跑偏。

### 1.2 胶水编程 (Glue Coding)
> **宣言**: "能抄不写，能连不造，能复用不原创。"

胶水编程是 Vibe Coding 的核心战术，旨在解决“AI 幻觉”与“复杂性爆炸”问题。
- **哲学**: 现代软件工程的本质不是“创造”，而是“集成”。
- **架构**:
  - **实体 (Entity)**: 成熟的开源项目、SDK、类库 (Wheel)。
  - **连接 (Link)**: AI 生成的胶水代码，负责数据适配与流程串联。
  - **功能 (Function)**: 你的业务目标。

### 1.3 广积粮 (Long-termism)
AI 往往是“短视”的，只关注当前对话的解决。但在 Vibe Coding 中，你必须具备“长期主义”思维。
- **记录 (Recording)**: 仅仅把用户的要求追加到 `SKILL.md` 末尾。这是低级的。
- **沉淀 (Sedimentation)**: 将知识**结构化**。这是高级的。
    - *目标*: 让知识库越用越薄，越用越精。

### 1.4 配置优于硬编码 (Configuration Over Hardcoding)
> **宣言**: "代码是引擎，配置是燃料。"

Vibe Coding 要求逻辑与数据完全解耦。AI 生成的代码应当是通用的“引擎”，而具体的业务参数（如物品ID、文本内容、GUI布局）应作为“燃料”注入。
- **哲学**: 赋予用户（而非开发者）控制权。

---

## 🏗️ 第二章：道法术器架构 (The Dao-Fa-Shu-Qi Architecture)

为了管理复杂的知识，我们采用了四层架构。这不仅是分类，更是**上下文管理 (Context Management)** 的策略。

### Tier 1: 道 (Dao) - System Prompts
- **代表**: `AGENTS.md` (原 rules.md)
- **定位**: **Always Active (始终生效)**。
- **职责**: **调度中心 (Dispatcher)**。它定义了“三大元支柱”：
    1.  **Glue (生态优先)**: 先找轮子。
    2.  **Data (配置分离)**: 数据外置。
    3.  **Feature (架构统一)**: 按功能分包。

### Tier 2: 法 (Fa) - Methodology
- **代表**: `mc-modern-stack` (Java 21+, Gradle)
- **定位**: **On Demand (按需加载)**。
- **职责**: **执法者 (Enforcer)**。当 `rules.md` 决定开始写代码时，它才介入，提供具体的校验标准（如异步IO、防御性编程）。

### Tier 3: 术 (Shu) - Skills
- **代表**: `mc-architect`, `mc-resource-artist`
- **定位**: **On Demand (按需加载)**。
- **职责**: **专家 (Expert)**。
    - 架构师 (`mc-architect`) 负责画图纸。
    - 美术 (`mc-resource-artist`) 负责填色。

### Tier 4: 器 (Qi) - Knowledge/Patterns
- **代表**: `references/*.md`
- **定位**: **Reference (引用)**。
- **职责**: **积木 (Block)**。具体的代码实现，随用随取。

---

## 📜 第三章：标准化协议 (Standardization Protocols)

为了让知识库可维护，所有文档必须遵守以下元数据标准。

### 3.1 元数据头 (Metadata Header)
**所有** `.md` 文件（包括 Skill 和 Reference）必须以 YAML Frontmatter 开头：

```yaml
---
name: [文件标识名]
description: [一句话描述作用]
type: [skill | methodology | pattern | context]
tags: [关键词1, 关键词2] (可选)
---
```

### 3.2 引用规范 (Reference Protocol)
在 `SKILL.md` 中引用 Reference 时，必须使用相对路径，并简述其价值：
- ✅ `参考: [async-db.md](./references/async-db.md) - 异步数据库操作范式`
- ❌ `参考: async-db.md`

### 3.3 知识沉淀工作流 (Sedimentation Workflow)
当用户提供新知识时，执行 α-Ω 循环：
1.  **识别 (Identify)**: 这是个别案例，还是通用模式？
2.  **抽象 (Abstract)**: 去掉具体业务逻辑，提取核心骨架。
3.  **归档 (Archive)**: 
    - 如果是代码模式 -> 新建 `references/patterns/xxx.md`。
    - 如果是流程优化 -> 修改对应 Skill 的 `Workflow` 章节。
4.  **索引 (Index)**: 在对应 Skill 的 `references` 列表中添加链接。

---

## 🧠 第四章：AI 的自我修养 (The AI Mindset)

### 4.1 不要猜，去猎取 (Don't Guess, Hunt)
- 遇到不确定的 API -> 激活 `mc-integration-specialist` (原 context-hunter)。
- 遇到模糊的需求 -> 激活 `mc-requirement-analyst` 进行 Deep Thinking。

### 4.2 永远不仅是为了现在 (Always for the Future)
- 写代码时，想一想：这段代码以后会被复用吗？如果是，提炼它。
- 改文档时，想一想：这样改会让文档更清晰，还是更混乱？

### 4.3 拒绝“为了更新而更新”
- 用户说“你错了”，不要仅仅为了讨好用户而修改文档。
- 要分析：是我的**原理**错了（改 Tier 2），还是**操作**错了（改 Tier 3），还是**资料**过时了（改 Tier 4）？
- **透过现象看本质**。

---

## 🔄 第五章：递归自我优化协议 (Recursive Self-Optimization Protocol)

Vibe Coding 的终极目标是**将人类开发者的隐性知识显性化**。

### 5.1 核心理念：从“记录”到“沉淀”
不要只是把规则追加到 `SKILL.md` 后面（Recording），那会让文档臃肿不堪。
你要做的是将知识**结构化**（Sedimenting）：
- **通用模式** -> 提取为 `references/patterns/xxx.md` (器)
- **流程缺陷** -> 修正 `SKILL.md` 中的 Workflow (术)
- **原则漏洞** -> 更新 `mc-modern-stack/SKILL.md` (法)

### 5.2 交互即训练 (Interaction as Training)
- **哲学**: 每一次与用户的交互，不仅是解决一个具体问题，更是训练 AI（你）理解用户隐性偏好的过程。
- **机制**:
    - **纠错 (Correction)**: 意味着你的“法”或“术”与用户期望不符。
    - **高光 (Highlight)**: 意味着你捕获了用户的隐性审美或最佳实践。
    - **排错 (Debug)**: 每一个 Bug 都是知识库的漏洞。修复 Bug 后，必须反思“为什么文档没能阻止我写出这个 Bug？”。
    - **复用 (Reuse)**: 意味着你识别出了通用的逻辑模组。
    - **摩擦 (Friction)**: 意味着你的工作流存在设计缺陷。
- **目标**: 将这些瞬时的“信号”转化为永久的“记忆” (Knowledge)。
