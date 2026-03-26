
# GPT 总结的 WPF 框架的 kerning 用法

本文记录我用 GPT 总结的 WPF 框架的 kerning 用法

<!--more-->


<!-- CreateTime:2026/03/26 07:11:44 -->

<!-- 不发布 -->

我问：

这是整个 WPF 框架仓库的源代码。请根据阅读仓库的代码回答我的问题。如何在 WPF 中实现 Kern 的排版效果。我期望能够直接使用 GlyphRun 的方式进行绘制。我需要知道如何排版测量获取 WPF 的多个字符之间的 Kern 值。请只完全依赖 WPF 框架，不依赖其他任何框架实现

GPT 回答：

可以，但要分清两件事：

1. `GlyphRun` **可以绘制**带 kerning 的结果。  
2. 但 `GlyphRun` **不会自己计算** kerning。你如果手工 `new GlyphRun(...)`，WPF 只吃你传进去的 `glyphIndices`、`advanceWidths`、`glyphOffsets`，不会再帮你做排版整形。

从仓库代码看，这一点很明确：

- `Typography` 里有公开的 `Kerning` 属性：`src/Microsoft.DotNet.Wpf/src/PresentationFramework/System/Windows/Documents/Typography.cs`
- 内部 OpenType feature tag 里把 kerning 映射到 `'kern'`：`src/Microsoft.DotNet.Wpf/src/DirectWriteForwarder/CPP/DWriteWrapper/DWriteFontFeatureTag.h`
- `TextLine` 公开了 `GetIndexedGlyphRuns()`，可以把 WPF 已经排版好的 `GlyphRun` 拿出来：`src/Microsoft.DotNet.Wpf/src/PresentationCore/System/Windows/Media/textformatting/TextLine.cs`
- `GlyphRun` 构造时只是保存 `advanceWidths/glyphOffsets`，没有公开 kerning 计算入口：`src/Microsoft.DotNet.Wpf/src/PresentationCore/System/Windows/Media/GlyphRun.cs`
- `GlyphTypeface` 公开的是 `CharacterToGlyphMap` 和单 glyph 的 `AdvanceWidths`，**没有公开“字偶间距表”查询 API**：`src/Microsoft.DotNet.Wpf/src/PresentationCore/System/Windows/Media/GlyphTypeface.cs`

## 结论

如果你要求：

- 只依赖 WPF
- 最终仍然用 `GlyphRun` 绘制
- 还要拿到 WPF 实际使用的 kerning 结果

那么正确路径是：

`TextFormatter` → `TextLine` → `GetIndexedGlyphRuns()` → 取出 `GlyphRun` → `DrawGlyphRun`

---

## 你不能直接做的事

仅靠公开的 `GlyphTypeface` API，**不能直接拿到某两个字符/两个 glyph 的原始 kern table 值**。

也就是说，这种思路做不到完整 kerning：

- `GlyphTypeface.CharacterToGlyphMap`
- `GlyphTypeface.AdvanceWidths`
- 手工 `new GlyphRun(...)`

因为这里拿到的只是**名义 advance**，不是排版引擎经过 kerning/GPOS 后的结果。

---

## 你应该怎么做

### 方案：先让 WPF 排版，再取回 `GlyphRun`

核心调用是：

```csharp
using System.Linq;
using System.Windows;
using System.Windows.Media;
using System.Windows.Media.TextFormatting;

// 建议用 Ideal，便于测量，避免 Display 模式像素取整影响
using TextFormatter formatter = TextFormatter.Create(TextFormattingMode.Ideal);

using TextLine line = formatter.FormatLine(
    textSource,
    0,
    double.PositiveInfinity,
    paragraphProperties,
    null);

GlyphRun glyphRun = line.GetIndexedGlyphRuns().Single().GlyphRun;

// 后续直接自己绘制
drawingContext.DrawGlyphRun(Brushes.Black, glyphRun);
```

这里的关键不是 `DrawGlyphRun`，而是 `line.GetIndexedGlyphRuns()`。  
这个 `GlyphRun` 已经是 WPF 内部排版后的结果，`AdvanceWidths` 和 `GlyphOffsets` 已经包含 kerning 影响。

---

## 如何“测量”多个字符之间的 kern 值

### 最稳妥的方法：做两次排版

同一段文本、同一字体、同一字号、同一 DPI、同一 `TextFormattingMode`：

- 一次 `Kerning = false`
- 一次 `Kerning = true`

然后比较两个 `GlyphRun` 的 glyph 位置差异。

### 为什么要这样做

因为：

- `GlyphTypeface.AdvanceWidths` 是名义宽度
- `GlyphRun.AdvanceWidths` 是排版后的实际宽度
- `Display` 模式还会有像素对齐/取整
- 某些字体会用 GPOS，不一定只是简单 pair kern

所以 **“WPF 最终到底用了多少 kern”**，最可靠的就是比较：

`withKerningGlyphRun` vs `withoutKerningGlyphRun`

---

## 位置计算方式

`GlyphRun` 的第 `i` 个 glyph 的 nominal X 原点可由前面所有 advance 累加得到，再叠加自己的 offset。

可用下面的帮助方法：

```csharp
static double[] GetGlyphOriginsX(GlyphRun run)
{
    int count = run.GlyphIndices.Count;
    double[] result = new double[count];
    double x = 0;

    for (int i = 0; i < count; i++)
    {
        double offsetX = 0;
        if (run.GlyphOffsets != null && run.GlyphOffsets.Count > i)
            offsetX = run.GlyphOffsets[i].X;

        result[i] = x + offsetX;
        x += run.AdvanceWidths[i];
    }

    return result;
}
```

然后比较：

```
double[] x0 = GetGlyphOriginsX(runWithoutKerning);
double[] x1 = GetGlyphOriginsX(runWithKerning);

// 第 i 个 glyph 因 kerning 产生的水平位移
double delta = x1[i] - x0[i];
```

这个 `delta` 就是 **WPF 实际应用后的 glyph 位移差**。

---

## 如果你只关心简单拉丁字偶

例如 `"AV"`、`"To"` 这种 1:1 映射场景，可以近似认为：

- 第二个 glyph 的原点变化量，就是这个字偶的 kerning 结果
- 或者总 advance 的差值，就是 pair 的压缩/拉开量

例如：

```csharp
double total0 = runWithoutKerning.AdvanceWidths.Sum();
double total1 = runWithKerning.AdvanceWidths.Sum();
double pairKern = total1 - total0;
```

对简单字偶通常成立。

---

## 但要注意这几个坑

### 1. ligature 会干扰“字偶值”
如果开启了连字，`"fi"` 可能直接变成一个 glyph。  
此时“两个字符之间的 kern 值”这个概念就不再稳定。

如果你想专门测 kerning，建议同时关闭这些特性：

- `StandardLigatures = false`
- `ContextualLigatures = false`
- `ContextualAlternates = false`

只保留：

- `Kerning = true`

---

### 2. 复杂脚本不一定存在“单个 pair kern”
阿拉伯文、印地文等脚本，WPF 可能通过更复杂的 shaping/positioning 处理。  
这时更准确的说法不是“字符对的 kern 值”，而是“排版后 glyph 的定位调整”。

---

### 3. `Display` 模式会引入取整
如果你在做精确测量，优先用：

```csharp
TextFormatter.Create(TextFormattingMode.Ideal)
```

否则 kerning 值里会混入像素对齐误差。

---

## 实际建议

如果目标是“最后自己用 `GlyphRun` 绘制”，推荐流程固定为：

1. 用 `TextFormatter` 格式化文本
2. 在 `TextRunTypographyProperties` 中设置 `Kerning = true`
3. 从 `TextLine.GetIndexedGlyphRuns()` 取出 `GlyphRun`
4. 直接 `DrawGlyphRun`
5. 如需 kern 数值，再与 `Kerning = false` 的结果做差

---

## 一句话结论

**只靠公开 WPF API，不能直接从 `GlyphTypeface` 读取原始 kern pair 值；但可以完全依赖 WPF 的 `TextFormatter` 先生成已应用 kerning 的 `GlyphRun`，再从 `GlyphRun.AdvanceWidths/GlyphOffsets` 中测量实际 kerning 结果。**

如果需要，下一条可以直接给你一份**最小可运行**的 `TextSource` / `TextRunProperties` / `TextParagraphProperties` 示例，专门用来输出带 kerning 的 `GlyphRun`。




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。