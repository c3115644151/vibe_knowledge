# DeepWiki 与 AI 辅助集成的局限性复盘 (Retrospective on DeepWiki Limitations)

## 📅 背景 (Context)
在集成 `CraftEngine` 插件时，我们需要获取自定义物品的本地化名称（`item-name`）。由于官方文档缺失，我们尝试使用 DeepWiki 读取仓库源码/文档以获取 API 指引。

## 🚫 失败现象 (Failure Symptoms)
1.  **API 幻觉 (Hallucination)**: DeepWiki 提供了看似合理的 API 方法（如 `CraftEngine.getApi().getItemName(id)`），但在 IDE 中无法解析，反编译 Jar 包后也未找到。
2.  **无效的自信 (False Confidence)**: AI 对不存在的 API 给出了详细的参数说明和使用示例，导致开发者误以为是依赖导入问题而非 API 不存在。
3.  **循环试错 (Looping)**: 当指出 API 不存在时，AI 只是换了一个同样不存在的方法名（如 `getCustomItem(id).getName()`），未能跳出思维定势。

## 🔍 根因分析 (Root Cause Analysis)

### 1. "Getter" 推理偏差 (The "Getter" Inference Bias)
*   **机制**: AI 看到配置文件中有 `item-name` 字段，根据 JavaBean 规范习惯性推断对应的 Java 类中一定有 `getItemName()` 方法。
*   **现实**: 许多 Minecraft 插件（尤其是闭源或混淆过的）直接使用 `Map<String, Object>` 存储数据，或者将数据封装在内部类中不对外暴露 Getter。

### 2. 源码与二进制的断裂 (Source vs Binary Gap)
*   **机制**: DeepWiki 读取的可能是 GitHub 上的开源版本（可能是 V2），而服务器运行的是闭源的高级版本（V3）。
*   **现实**: 插件作者经常重构 API，或者开源版本仅为 API Stub，并不包含核心逻辑。AI 无法验证 Git 仓库代码与实际 Jar 包的一致性。

### 3. 上下文污染 (Context Contamination)
*   **机制**: AI 训练数据中包含大量类似的插件（如 ItemsAdder, MMOItems）。
*   **现实**: 当特定插件的知识稀缺时，AI 会无意识地借用其他同类插件的 API 模式（如 `ItemsAdder.getCustomItem()`）并套用到当前插件上。

## 🧠 教训与改进 (Lessons Learned)

### 1. 验证优先 (Verification First)
*   **Action**: 在尝试 AI 提供的 API 之前，先使用 IDE 的自动补全或反编译工具（如 JD-GUI）验证方法是否存在。
*   **Rule**: 如果 IDE 提示 "Cannot resolve method"，不要假设是依赖问题，首先假设是 **API 根本不存在**。

### 2. 快速熔断 (Fast Fail)
*   **Action**: 连续 2 次 API 调用失败后，立即停止询问 API。
*   **Rule**: 转向 **"数据优先"** 思维 —— 不问 "怎么调用 API"，而问 "数据存在哪个文件里"。

### 3. 降级策略 (Fallback Strategy)
*   **Action**: 当 API 不可用时，毫不犹豫地采用 **配置劫持 (Config Hijack)** 模式。
*   **Mindset**: 此时文件系统操作虽然"暴力"，但比不存在的 API 更可靠。
*   

## 📝 结论 (Conclusion)
DeepWiki 和 LLM 是强大的**推断引擎**，而非**搜索引擎**。在面对闭源或文档稀缺的专有软件集成时，必须保持怀疑态度，将 AI 的建议视为"伪代码"而非"可执行代码"。

**最终解决方案**: 放弃寻找 API，直接读取 `plugins/CraftEngine/resources` 下的 YAML 文件，构建自己的数据索引。
