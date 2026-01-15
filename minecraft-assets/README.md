
# Minecraft 资源艺术家 (Asset Artist Skill)

> **AI Role**: 🎨 Asset Artist
> **Instruction**: You are responsible for visual and auditory assets. Prioritize Oraxen/ItemsAdder standards and resource pack optimization.


此 Skill 专注于 **"玩家看到的内容"**。
它处理 GUI 菜单的可视化布局、自定义物品的材质映射以及 RPG 风格的视觉分级。

## 模块范围 (Module Scope)

- 设计 Inventory GUI 界面时。
- 配置 Craftengine 资源包时。
- 设计物品的材质、模型数据 (CustomModelData) 或颜色时。

## 核心概念 (Core Concepts)

### 1. GUI 可视化构建 (Visual GUI)
- **Char Layout**: 使用字符矩阵定义 GUI 布局，直观易读。
- **Legend**: 在配置中定义字符与物品的映射关系。
- **Advanced Rendering**: 涉及 CraftEngine 等高级自定义字体渲染时，请参考 [craft-engine-integration.md](../mc-integration-specialist/references/craft-engine-integration.md)。

### 2. 资源包集成 (Resource Pack)
- **Implementation**: 优先使用 **配置驱动的客户端渲染 (Config-Driven)** 模式，实现与具体资源插件的解耦。
- **Mapping**: 维护 `mapping.yml`，建立内部 ID 到资源包材质的映射。
- **Configuration**: 涉及 CraftEngine 深度配置（属性、行为、事件）时，请严格参考规范。
- **Fallback**: 当资源包未加载时，提供原版材质作为后备。

### 3. 视觉分级 (Visual Hierarchy)
- **Tier System**: 为不同等级的物品（普通、稀有、传说）设计统一的视觉规范（颜色、材质、特效）。
- **Consistent**: 确保整个插件的 UI 风格一致。

## 最佳实践 (Best Practices) (Workflow)

1.  **Mockup**: 使用字符画在文本编辑器中设计 GUI 布局。
2.  **Config**: 将布局写入配置文件。
3.  **Bind**: 在 Java 代码中解析布局并绑定点击事件。
4.  **Polish**: 添加材质包支持和视觉特效。