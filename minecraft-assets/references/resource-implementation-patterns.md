# 资源包实现模式 (Resource Implementation Patterns)

本文档记录了在 Minecraft 插件开发中，将自定义资源（贴图、模型）与代码逻辑连接的几种常见实现模式。

## 1. 模式 A：配置驱动的客户端渲染 (Config-Driven Client Rendering) - ✨ 最佳实践

此模式采用**完全解耦**的思路。服务器端插件不需要知道“物品是由 ItemsAdder 还是 Oraxen 提供的”，也不需要读取本地的资源配置文件。插件只需负责向客户端发送带有正确 `CustomModelData` 的原版物品数据包。

### 核心逻辑
- **数据源**: `config.yml` (或其他插件配置)。
- **渲染**: 客户端资源包 (Resource Pack)。
- **桥梁**: `Material` + `CustomModelData`。

### 优缺点
- ✅ **解耦**: 插件不依赖 ItemsAdder/Oraxen API，甚至不需要这些插件安装在服务端。
- ✅ **灵活**: 支持在线/云端资源包 (Server Resource Pack)，贴图文件无需存在于服务端磁盘。
- ✅ **轻量**: 减少了服务器启动时的挂钩 (Hook) 和注册表查找开销。
- ❌ **验证**: 插件无法得知该 ID 对应的贴图是否真实存在（“盲发”）。

### 代码范式

**config.yml**:
```yaml
gui_assets:
  next_page:
    material: PAPER
    custom_model_data: 50004
    name: "&e下一页"
```

**Java 实现**:
```java
public ItemStack getGuiAsset(String key) {
    String path = "gui_assets." + key;
    // 1. 读取配置
    int cmd = config.getInt(path + ".custom_model_data");
    Material mat = Material.matchMaterial(config.getString(path + ".material"));
    
    // 2. 组装物品 (无需查询外部 API)
    ItemStack item = new ItemStack(mat);
    ItemMeta meta = item.getItemMeta();
    meta.setCustomModelData(cmd); // 核心：注入数据
    item.setItemMeta(meta);
    
    return item;
}
```

---

## 2. 模式 B：钩子注册表查找 (Hook-Based Registry Lookup) - 传统方案

此模式依赖外部资源管理插件（ItemsAdder/Oraxen）提供的 API。插件向管理器询问：“请给我 ID 为 `namespace:item_id` 的物品堆”。

### 核心逻辑
- **数据源**: ItemsAdder/Oraxen 的内部注册表。
- **依赖**: 必须安装对应的资源管理插件。

### 优缺点
- ✅ **验证**: 如果物品 ID 不存在或配置错误，API 会返回 null，便于排查问题。
- ✅ **同步**: 自动获取资源插件定义的属性（如 DisplayName, Lore, NBT）。
- ❌ **耦合**: 强依赖特定插件 API。
- ❌ **本地限制**: 服务器必须加载了对应的资源配置，无法仅靠客户端资源包工作。

### 代码范式

**Java 实现**:
```java
public ItemStack resolveItem(String id) {
    // ItemsAdder 查找
    if (Bukkit.getPluginManager().isPluginEnabled("ItemsAdder")) {
        CustomStack stack = CustomStack.getInstance(id);
        if (stack != null) return stack.getItemStack();
    }
    // Oraxen 查找
    if (Bukkit.getPluginManager().isPluginEnabled("Oraxen")) {
        OraxenItems.getItemById(id).build();
    }
    return null;
}
```

---

## 3. 模式 C：混合回退策略 (Hybrid Fallback Strategy) - 鲁棒方案

结合上述两种模式，优先尝试从注册表获取（以获得完整的属性支持），如果失败或未安装插件，则回退到配置驱动模式或原版物品。

### 核心逻辑
1. 尝试 `ItemsAdder/Oraxen` Hook。
2. 尝试 `Material` + `CustomModelData` (Config)。
3. 回退到纯 `Material` (Vanilla)。

### 代码范式

```java
public ItemStack createIcon(Recipe recipe) {
    ItemStack item = null;

    // 1. 尝试 Hook (Pattern B)
    if (recipe.getCustomId() != null && Hooks.isLoaded()) {
        item = Hooks.getItem(recipe.getCustomId());
    }

    // 2. 尝试 Config/ModelData (Pattern A)
    if (item == null && recipe.getModelData() != 0) {
        item = new ItemStack(recipe.getMaterial());
        ItemMeta meta = item.getItemMeta();
        meta.setCustomModelData(recipe.getModelData());
        item.setItemMeta(meta);
    }

    // 3. 原版回退
    if (item == null) {
        item = new ItemStack(recipe.getMaterial());
    }

    return item;
}
```

---

## 4. 模式 D：原生框架配置 (Framework-Native Configuration) - CraftEngine 专属

对于深度集成了 **CraftEngine** 的项目，物品不仅仅是“材质+模型”，而是包含了行为 (`behaviors`) 和事件 (`events`) 的完整方块/实体定义。

### 核心逻辑
- **数据源**: CraftEngine 自身的 `configuration/blocks` 或 `configuration/items` YAML 文件。
- **集成**: 插件通过 CE API (`CraftEngineHook`) 交互，或让 CE 自动接管逻辑。
- **优势**: 无需编写 Java 代码即可实现复杂的方块逻辑（如含水、右键交互、掉落表）。

### 参考
详情请查阅深度配置手册：[craft-engine-configuration.md](./craft-engine-configuration.md)

---

## 附录：ItemsAdder 资源包结构参考 (Appendix: IA Pack Structure)

如果项目决定采用 ItemsAdder 进行资源管理（Pattern B），推荐采用以下目录结构与配置规范。

### 1. 目录结构规范
```
plugins/ItemsAdder/contents/
└── [namespace_pack] (e.g., server_pack)
    ├── configs/
    │   └── [plugin_name].yml       # 物品定义文件
    └── resourcepack/
        └── assets/
            └── minecraft/
                ├── models/
                │   └── custom/
                │       └── [plugin_name]/  # JSON 模型文件
                └── textures/
                    └── custom/
                    └── [plugin_name]/  # PNG 贴图文件
```

### 2. 配置文件模板 (Config Template)
建议使用 `generate: false` 配合 `model_path`，以获得对模型的最大控制权。

```yaml
info:
  namespace: [plugin_name]

items:
  [item_id]:
    display_name: "物品名称"
    resource:
      material: [VANILLA_MATERIAL]
      generate: false
      model_path: minecraft:custom/[plugin_name]/[item_id]
```
