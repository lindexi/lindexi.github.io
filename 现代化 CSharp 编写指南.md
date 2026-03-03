# 现代化 C# 编写指南

这是给 AI 看的提示词内容

<!--more-->
<!-- 不发布 -->

1. 网络层统一使用 `HttpClient` + `HttpRequestMessage`，避免 `HttpWebRequest`。
2. JSON 处理优先使用 `System.Text.Json`，并尽量配合源生成上下文（`JsonSerializerContext`）提升 AOT 兼容性。
  - 将所有 `Newtonsoft.Json` 的功能替换为调用 `System.Text.Json` 实现
  - 禁止通过伪造兼容层保留 `Newtonsoft` 特性；应直接迁移为 `System.Text.Json` 特性与 API
3. 对关键入参增加早期校验（如 `ArgumentNullException.ThrowIfNull` / `ArgumentException.ThrowIfNullOrWhiteSpace`），减少隐式空引用风险。
4. 常用集合初始化时尽量预估容量（如 `new List<T>(count)`、`new Dictionary<TKey, TValue>(capacity)`），减少扩容开销。
5. 字符串拼接优先使用内插字符串（`$"..."`），替代 `string.Format`，可读性更高。
6. 属性、方法、字段、类型名等的命名需要符合 C# 命名规范，例如：
  - 属性、方法、类型名应该使用 PascalCasing 命名风格
  - 局部变量应该使用 camelCase 命名风格
  - 字段应该使用 `_camelCase` 风格，而且尽可能使用 `readonly ` 标记只读。对于 `static readonly` 来说，应该视为常量，应该采用 PascalCasing 命名风格
7. 最好不要公开字段，请封装为属性进行公开
8. 考虑可空。但可以对进行网络请求的纯数据结构放宽要求
9. 改造时，考虑 API 兼容性。旧 API 的命名保留兼容入口（`[Obsolete]`），新增规范命名 API 并逐步迁移