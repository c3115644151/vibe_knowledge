# [Project Name] Technical Manual (MANUAL.md)

## 1. 系统概述 (System Overview)
*[简要描述系统是如何运作的，核心玩法闭环是什么]*
例如：本插件通过 PDC 存储肥力数据，监听植物生长事件，根据基因型动态计算生长倍率，实现了...

---

## 2. 核心架构映射 (Core Architecture Map)
*快速定位代码与功能的对应关系。*

| 模块 | 关键类/文件 | 职责 (Responsibility) |
| :--- | :--- | :--- |
| **[模块名]** | `ManagerClass.java` | [描述核心职责] |
| **[模块名]** | `ListenerClass.java` | [描述监听了什么事件，处理了什么逻辑] |
| **[模块名]** | `ObjectData.java` | [描述数据结构定义] |

---

## 3. 机制详解 (Mechanics & Algorithms)
*将代码逻辑“翻译”为人话，这是本文档的核心价值。*

### 3.1 [子系统名称] (e.g., 生长效率计算)
- **公式**: `Efficiency = Base * (1 + Fertilizer) * Genetics`
- **逻辑流程**:
    1.  监听 `BlockGrowEvent`。
    2.  读取 PDC 中的肥力值。
    3.  读取物品手中的基因数据。
    4.  ...
- **边界情况 (Edge Cases)**:
    - 若肥力 < 0，生长会被取消。
    - 若使用了骨粉，跳过肥力检查。

### 3.2 [子系统名称] (e.g., 基因突变)
- **触发条件**: ...
- **概率模型**:
    - 基础概率: 5%
    - 环境加成: +1% (在地狱)
- **状态流转**: `Type A` -> `Type B` -> `Type C`

---

## 4. 操作指南 (Operations)
*面向管理员或高级玩家的指令与配置说明。*

### 4.1 指令系统 (Commands)
- `/cmd get <item>`: 获取测试物品。
- `/cmd debug`: 开启调试模式，显示 ActionBar 数据。

### 4.2 调试工具 (Debug Tools)
- **调试棒**: 右键方块，显示 [Key: Value] 数据。
- **分析仪**: 查看隐藏的 NBT 数据。

---

## 5. API 与 事件 (API & Events)
*面向其他开发者的接口说明。*

### 5.1 核心事件
- `CustomEvent`: [触发时机]
- `Cancellable`: [是/否]

### 5.2 数据接口
- `Manager.getData(Location)`: 获取指定位置的数据快照。
