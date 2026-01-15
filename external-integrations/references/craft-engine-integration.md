# CraftEngine 集成与高级 GUI 渲染指南

本文档总结了与 **CraftEngine (CE)** 进行深度集成时的核心挑战、解决方案及最佳实践，特别是针对自定义 GUI 渲染与资源包重载的兼容性处理。

## 1. 核心挑战：资源重载与缓存失效 (The Reload Problem)

### 现象
当管理员执行 `/ce reload all` 重载 CraftEngine 资源包后，其他插件已加载的 GUI 界面中，引用自 CE 的物品图标会变成空气 (AIR)、丢失材质或 Lore 显示异常。

### 根因分析 (Root Cause)
1.  **注册表重建**: CE 在重载时会完全清除并重建内存中的物品注册表 (`ItemRegistry`)。
2.  **对象脱节**: 插件通常在 `onEnable` 或配置加载阶段调用 CE API 获取 `ItemStack` 并将其缓存在静态变量中。这些缓存的 `ItemStack` 对象持有的是**旧注册表**的引用或 NBT 数据。
3.  **失效**: 当新注册表建立后，旧的 `ItemStack` 无法正确映射到新的内部 ID 或模型数据，导致渲染失败。

### 解决方案：懒加载 (Lazy Loading)

**核心原则**: **严禁缓存 CE 的 `ItemStack` 对象。仅缓存 ID (String)。**

*   **Anti-Pattern (错误)**:
    ```java
    // 错误：在加载时缓存 ItemStack
    public void load() {
        this.icon = CraftEngineHook.getItem("my_icon"); 
    }
    // 使用时直接用旧对象
    inv.setItem(0, this.icon); 
    ```

*   **Best Practice (正确)**:
    ```java
    // 1. 定义：仅存储配置数据
    public class IconDef {
        String ceId;
        String fallbackMaterial;
        // ...
    }
    
    // 2. 实时构建：每次使用时重新获取
    public ItemStack getItem() {
        if (ceId != null) {
            // 实时向 CE 请求最新对象
            ItemStack item = CraftEngineHook.getItem(ceId);
            if (item != null) return item;
        }
        return new ItemStack(Material.valueOf(fallbackMaterial));
    }
    ```

---

## 2. 高级 GUI 渲染 (Advanced GUI Visualization)

CE 提供了强大的字体图像 (`FontImage`) 功能，允许通过 Unicode 字符在 GUI 标题中渲染高清纹理。

### 2.1 负空间布局 (Negative Space Layout)

要在 GUI 中完美对齐背景图，必须利用负空间字符消除容器默认边距。

**标准公式**:
```
Title = <shift:-Offset> + <image:background_id> + <shift:-Width> + <Content_Components>
```

*   **Offset**: 通常为 `-8` (消除左侧默认边距)。
*   **Background**: 渲染背景大图。
*   **Width**: 背景图的像素宽度 (用于将光标拉回左侧，以便渲染后续文本/图标)。

### 2.2 动态图标与状态机

利用 CE 的 `<image:id>` 标签，可以在标题中实现动态 UI 更新（如进度条、温度计）。

*   **实现**:
    1.  在配置中定义布局模板：`Layout: <image:bg><shift:-100>%progress%`
    2.  代码中根据状态替换占位符：
        ```java
        String progressIconId = isCooking ? "progress_bar_50" : "transparent";
        String title = layout.replace("%progress%", "<image:" + progressIconId + ">");
        ```
    3.  **注意**: 标题长度有限制。尽量使用预计算的 Unicode 字符而非长标签字符串。

### 2.3 透明占位符 (Invisible Placeholders)

为了防止 GUI 槽位中的真实物品（如点击触发器）遮挡背景图，需要使用透明物品。

*   **规范**:
    *   材质: `PAPER`。
    *   模型: 指定一个全空的 16x16 像素 PNG。
    *   名称: `<!i>` (CE 提供的隐藏名称标签) 或空字符。

---

## 4. 架构边界：物理 vs 逻辑 (Physical vs Logical Boundary)

在与 CE 集成时，必须清晰界定 **“什么是 CE 管的”** 和 **“什么是插件管的”**，避免越界导致的逻辑冲突或无效开发。

### 4.1 权力边界表 (The Power Boundary)

| 维度 | CraftEngine (Physical) | Your Plugin (Logical) | 典型案例 |
| :--- | :--- | :--- | :--- |
| **存在性** | **定义者 (Definer)** | **使用者 (Consumer)** | CE 定义 `rice` 的 ID 和模型；插件引用该 ID 进行种植。 |
| **物理属性** | **完全接管 (Owner)** | **只读 (Read-Only)** | 硬度、**含水 (Waterlogged)**、光照、碰撞箱。**必须在 CE 配置文件中修改，插件代码无权干涉。** |
| **视觉表现** | **渲染者 (Renderer)** | **触发者 (Trigger)** | CE 负责加载贴图、模型变种；插件负责决定“当前是第几阶段”。 |
| **交互逻辑** | **基础 (Basic)** | **核心 (Core)** | CE 处理简单的右键打开 GUI；插件处理复杂的生长算法、多方块机器判定。 |
| **数据存储** | **状态 (State)** | **业务 (Business)** | CE 存 `BlockState`；插件存 `PDC`、数据库、机器库存。 |

### 4.2 反模式 (Anti-Patterns)

*   **越权物理修改**: 试图通过监听 `BlockPhysicsEvent` 强行阻止水流，而不在 CE 配置中定义 `fluid-state: water`。
    *   *后果*: 服务端底层物理引擎与插件逻辑打架，导致“锁水”或幽灵方块。
*   **重复造轮子**: 在插件代码中手写 NMS 逻辑去修改方块属性。
    *   *后果*: 破坏 CE 的状态管理，导致重载后属性丢失。
*   **配置错位**: 在插件的 `config.yml` 中定义方块的硬度或模型路径。
    *   *后果*: 无效配置，CE 根本不读取这些文件。

### 4.3 最佳实践 (Best Practices)

1.  **遇事不决查边界**: 遇到物理/视觉问题（如模型不对、水流不通、光照异常），先查 CE 配置 (`items.yml`/`blocks.yml`)。
2.  **遇事不决查逻辑**: 遇到交互/数据问题（如机器不工作、生长停止、数据丢失），先查插件代码。
3.  **配置即协议**: 将 CE 的配置文件视为插件与底层的 **通信协议**。如果需要改变底层行为（如让方块含水），必须修改协议（配置文件），而不是试图在应用层（插件代码）进行 Hack。

---

## 3. 插件对接规范 (Integration Protocol)

### 3.1 反射 Hook (Reflection)

为了避免硬依赖导致插件无法在无 CE 的环境启动，必须使用反射进行隔离。

**关键类与方法**:
| 目标类 | 关键方法 | 用途 |
| :--- | :--- | :--- |
| `CraftEngineItems` | `get(Key)` / `byId(Key)` | 获取物品堆 |
| `FontManager` | `codepointByImageId(Key)` | 获取图片的 Unicode 字符 |
| `Key` | `of(String)` | 构建 NamespacedKey |

### 3.2 资源包结构 (Resource Pack Structure)

建议插件内置 CE 资源包结构，方便用户直接合并。

```
plugins/CraftEngine/resources/[namespace]/
├── configuration/
│   ├── items.yml      # 物品定义
│   └── blocks.yml     # 方块定义
└── resourcepack/
    └── assets/
        └── [namespace]/
            ├── textures/
            └── models/
```

### 3.3 材质迁移 (Migration)

从 ItemsAdder (IA) 迁移到 CE 时：
1.  **配置转换**: 将 IA 的 `.yml` 配置转换为 CE 的 `items.yml` 格式。注意属性名差异 (`display_name` -> `name`, `resource.model_path` -> `model`)。
2.  **资源移动**: 将材质和 JSON 模型文件移动到 CE 对应目录。
3.  **代码适配**: 更新 Hook 逻辑，优先检查 CE ID。

---

## 4. 最佳实践清单 (Checklist)

- [ ] **Lazy Loading**: 确认所有 CE 物品均为实时获取，无静态缓存。
- [ ] **Fallback**: 确认当 CE 未安装或物品 ID 错误时，有原版材质回退，不会导致 NPE。
- [ ] **Reload Test**: 执行 `ce reload` 后，检查 GUI 贴图和 Lore 是否依然正常。
- [ ] **Namespace**: 所有 ID 必须带命名空间 (e.g. `cuisinefarming:pot_bg`) 防止冲突。
- [ ] **Transparent Item**: 检查 GUI 点击槽位是否使用了透明物品占位。
