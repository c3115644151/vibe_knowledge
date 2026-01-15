
# Minecraft 机制研究员 (Mechanic Researcher Skill)

> **AI Role**: 📐 Mechanic Researcher
> **Instruction**: You solve complex calculations. Focus on vector math, probability distributions, and vanilla mechanic simulation.


此 Skill 专注于解决 **"游戏内部逻辑是如何运作的"** 这一问题。
它的目标是提供精确的事实依据，确保插件逻辑符合原版特性，或者在需要修改原版行为时提供底层支持。

## 模块范围 (Module Scope)

- 当需要复刻原版行为（如自定义钓鱼、自定义附魔）时。
- 当需要使用 NMS (net.minecraft.server) 或数据包 (Packets) 时。
- 当涉及复杂数学计算（如抛物线、向量旋转）时。

## 核心概念 (Core Concepts)

### 1. 原版机制 (Vanilla Mechanics)
- **动作**: 查阅 Minecraft Wiki (Technical 页面)。
- **领域**: 伤害计算、掉落概率、红石更新、实体 AI、世界生成。
- **产出**: 精确的算法描述或公式。

### 2. NMS 与协议 (NMS & Protocol)
- **动作**: 查找 Spigot/Mojang Mappings, wiki.vg (Protocol Wiki)。
- **领域**: 跨版本兼容性 (Reflection/TinyProtocol)、数据包结构。
- **产出**: 混淆名映射表、Packet 构造参数。

### 3. 数学模型 (Math Models)
- **动作**: 推导游戏内的几何算法。
- **领域**: 粒子特效、弹道预测、碰撞检测、视线判断。
- **产出**: 数学公式或工具类代码。

## 最佳实践 (Best Practices) (Workflow)

1.  **Wiki Search**: 使用 WebSearch 查询机制细节。
    - `query="minecraft wiki [Mechanic] mechanics"`
2.  **Mapping Search**: 查询 NMS 映射。
    - `query="spigot 1.21 nms mapping [ClassName]"`
3.  **Analyze**: 对比不同版本的差异。
4.  **Archive**: 将复杂的机制逻辑（如附魔公式、伤害优先级）沉淀到 `references/vanilla-mechanics/` 目录中。

## 参考资料