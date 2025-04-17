---
title: WPF 测试 GlyphTypeface 的 Baseline 行为
description: 本文将对 WPF 进行 GlyphTypeface 的 Baseline 行为测试。经过测试发现行为非常符合预期，这个值乘以字号就是基线
tags: WPF
category: 
---

<!-- CreateTime:2025/01/08 07:01:36 -->

<!-- 发布 -->
<!-- 博客 -->

本文内容里面只给出关键代码片段，如需要全部的项目文件，可到本文末尾找到本文所有代码的下载方法

前置博客： [WPF 简单聊聊如何使用 DrawGlyphRun 绘制文本](https://blog.lindexi.com/post/WPF-%E7%AE%80%E5%8D%95%E8%81%8A%E8%81%8A%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-DrawGlyphRun-%E7%BB%98%E5%88%B6%E6%96%87%E6%9C%AC.html )
<!-- [WPF 简单聊聊如何使用 DrawGlyphRun 绘制文本 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/15388005.html ) -->

大飞哥来问我关于文本行距和基线问题，在之前某只不远透露姓名的牛写了一段有趣的代码，设定了行距计算里面包含 1/5 的魔法数字。我开始猜测是基线计算的问题，结果一顿计算发现数据差异过大，没有解决开始的问题，只好将我测试的 GlyphTypeface 的 Baseline 行为记录

在 WPF 里面，可以通过 FontFamily 根据字体名字符串获取到 GlyphTypeface 对象，大概的代码如下

```csharp
        var fontFamily = new FontFamily("微软雅黑");

        Typeface typeface = fontFamily.GetTypefaces().First();

        var success = typeface.TryGetGlyphTypeface(out GlyphTypeface glyphTypeface);
        if (!success)
        {
            Debug.Fail("微软雅黑字体找不到");
        }
```

我尝试绘制一段文本，内容是“文本测试afgjqiWHXx”

这段文本的特征是中英文混排，且英文字符有穿越基线的字符

我尝试按照 [WPF 简单聊聊如何使用 DrawGlyphRun 绘制文本](https://blog.lindexi.com/post/WPF-%E7%AE%80%E5%8D%95%E8%81%8A%E8%81%8A%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-DrawGlyphRun-%E7%BB%98%E5%88%B6%E6%96%87%E6%9C%AC.html ) 博客提供的方法构建 GlyphRun 进行绘制，代码如下

```csharp
        var fontSize = 30;

        var text = "文本测试afgjqiWHXx";
        var glyphIndexList = new List<GlyphInfo>();

        for (var i = 0; i < text.Length; i++)
        {
            var codePoint = (int) text[i]; // 这里的 Code Point 没有处理 Emoji 的高低代理字符
            if (glyphTypeface.CharacterToGlyphMap.TryGetValue(codePoint, out var glyphIndex))
            {
                var width = glyphTypeface.AdvanceWidths[glyphIndex] * fontSize;
                var height = glyphTypeface.AdvanceHeights[glyphIndex] * fontSize;
                glyphIndexList.Add(new GlyphInfo(glyphIndex, width, height));
            }
            else
            {
                // 进入字体回滚
            }
        }

        var pixelsPerDip = (float) VisualTreeHelper.GetDpi(this).PixelsPerDip;

        var baseline = glyphTypeface.Baseline * fontSize;

        var location = new Point(0, baseline);
        drawingContext.PushGuidelineSet(new GuidelineSet([0], [baseline]));

        var defaultXmlLanguage =
            XmlLanguage.GetLanguage(CultureInfo.CurrentUICulture.IetfLanguageTag);

        var glyphRun = new GlyphRun
        (
            glyphTypeface,
            bidiLevel: 0,
            isSideways: false,
            renderingEmSize: fontSize,
            pixelsPerDip: pixelsPerDip,
            glyphIndices: glyphIndexList.Select(t => t.GlyphIndex).ToList(),
            baselineOrigin: location, // 设置文本的偏移量
            advanceWidths: glyphIndexList.Select(t => t.AdvanceWidth).ToList(), // 设置每个字符的字宽，也就是字号
            glyphOffsets: null, // 设置每个字符的偏移量，可以为空
            characters: text.ToCharArray(),
            deviceFontName: null,
            clusterMap: null,
            caretStops: null,
            language: defaultXmlLanguage
        );

        drawingContext.DrawGlyphRun(Brushes.Black, glyphRun);
```

我尝试使用 DrawLine 将 baseline 的值绘制出来，代码如下

```csharp
        drawingContext.DrawLine(new Pen(Brushes.Black,1), new Point(0, baseline), new Point(300, baseline));
```

运行代码，可见画出来的线条就刚好是文本的基线，非常正确

<!-- ![](image/WPF 测试 GlyphTypeface 的 Baseline 行为/WPF 测试 GlyphTypeface 的 Baseline 行为0.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202501/1080237-20250108070307796-1801118592.png)

如此可证明将 GlyphTypeface 的 Baseline 属性乘以字号就是文本字符的基线

那 GlyphTypeface 的 Baseline 属性和 FontFamily 的有什么不同？绝大部分字体这两个属性都是相同的，但是由于字体可能存在加粗斜体等，为了更好的视觉呈现，确实存在不同的情况。有些 GlyphTypeface 和 FontFamily 存在不相同的 Baseline 属性

对于最终渲染来说，就应该获取对应的 GlyphTypeface 的基线。但由于 GlyphTypeface 和 FontFamily 的基线基本相差不大，也可以放心直接就用 FontFamily 的基线就好。毕竟在很多文本排版里面，是不期望只是加粗或带斜体一下，就让字体在行内上浮下沉

对于一些字体设计师来说，会特别修改加粗的基线，虽然从排版数值上让字体下沉，但视觉效果却刚好看起来是顺着的。从这个思路上说，拿 GlyphTypeface 的基线是更加正确的

通过 FormattedText 获取到的 Baseline 基本等于 FontFamily 的 Baseline 乘以字号，可能会和 GlyphTypeface 的不相同，如以下代码片段

```csharp
                    var text = "1";
                    var fontSize = 30;

                    var formattedText = new FormattedText(text, CultureInfo.CurrentCulture,
                        System.Windows.FlowDirection.LeftToRight, typeface, fontSize, Brushes.Black, pixelsPerDip);
                    var sameGlyphTypefaceAndFormattedText = Math.Abs(formattedText.Baseline - glyphTypeface.Baseline * fontSize) < 0.01; 
                    var sameFontFamilyAndFormattedText = Math.Abs(formattedText.Baseline - fontFamily.Baseline * fontSize) < 0.01;
```

在我设备上的所有字体都是 `sameFontFamilyAndFormattedText` 为 true 的值。即如果只是想通过 FormattedText 获取基线，那完全和使用 FontFamily 的 Baseline 乘以字号是等价的

通过阅读 WPF 源代码，可以理解到 FormattedText 的 Baseline 为什么和 FontFamily 几乎等价，原因是 FormattedText 的 Baseline 是从首行 TextLine 的 Baseline 获取到的。在 SimpleTextLine 类型里面的 Baseline 属性定义如下

```csharp
SimpleRun run = (SimpleRun)runs[count];
var realAscent = Math.Max(realAscent, run.Baseline);
_baselineOffset = formatter.IdealToReal(TextFormatterImp.RealToIdeal(realAscent), PixelsPerDip);

        /// <summary>
        /// Client to get the distance from top to baseline of this text line
        /// </summary>
        public override double Baseline
        {
            get { return _baselineOffset; }
        }
```

而 SimpleRun 的 Baseline 定义如下

```csharp
        internal double Baseline
        {
            get
            {
                if (Ghost || EOT)
                    return 0;

                return TextRun.Properties.Typeface.Baseline(TextRun.Properties.FontRenderingEmSize, 1, _pixelsPerDip, _textFormatterImp.TextFormattingMode);
            }
        }
```

可见是进入到 Typeface 的 Baseline 方法里面

```csharp
    public class Typeface
    {
        internal double Baseline(double emSize, double toReal, double pixelsPerDip, TextFormattingMode textFormattingMode)
        {
            return CachedTypeface.FirstFontFamily.Baseline(emSize, toReal, pixelsPerDip, textFormattingMode);            
        }
    }
```

如此可以看到，绕了一圈还是回到了 IFontFamily 的 Baseline 方法。来对比一下 FontFamily 类型的 Baseline 属性，以及 IFontFamily 接口的 PhysicalFontFamily 实现的 Baseline 方法

```csharp
    public class FontFamily
    {
        internal IFontFamily FirstFontFamily { get; }

        public double Baseline
        {
            get
            {
                return FirstFontFamily.BaselineDesign;
            }

            set
            {
                VerifyMutable().SetBaseline(value);
            }
        }
    }

    internal sealed class PhysicalFontFamily : IFontFamily
    {
        double IFontFamily.BaselineDesign
        {
            get
            {
                return ((IFontFamily)this).Baseline(1, 1, 1, TextFormattingMode.Ideal);
            }
        }

        double IFontFamily.Baseline(double emSize, double toReal, double pixelsPerDip, TextFormattingMode textFormattingMode)
        {
            if (textFormattingMode == TextFormattingMode.Ideal)
            {
                return emSize * _family.Metrics.Baseline;
            }
            else
            {
                double realEmSize = emSize * toReal;
                return TextFormatterImp.RoundDipForDisplayMode(_family.DisplayMetrics((float)(realEmSize), checked((float)pixelsPerDip)).Baseline * realEmSize, pixelsPerDip) / toReal;
            }
        }
    }
```

如此可见 FormattedText 走的逻辑和 FontFamily 基本相同，只有一些数值上的差异而已

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/bbde4f7c3873aac83c466762ac12ae37a3dccfa4/WPFDemo/LahallgucheHichawaki) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bbde4f7c3873aac83c466762ac12ae37a3dccfa4/WPFDemo/LahallgucheHichawaki) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bbde4f7c3873aac83c466762ac12ae37a3dccfa4
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bbde4f7c3873aac83c466762ac12ae37a3dccfa4
```

获取代码之后，进入 WPFDemo/LahallgucheHichawaki 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

附录：

以下是我使用如下代码跑出来的基线集合

```csharp
        var pixelsPerDip = VisualTreeHelper.GetDpi(this).PixelsPerDip;

        foreach (FontFamily? fontFamily in System.Windows.Media.Fonts.SystemFontFamilies)
        {
            if (!fontFamily.FamilyNames.TryGetValue(XmlLanguage.GetLanguage("zh-CN"),out var name))
            {
                name = fontFamily.Source;
            }
            foreach (var typeface in fontFamily.GetTypefaces())
            {
                var typefaceName = typeface.FaceNames.First().Value;
                if (typeface.TryGetGlyphTypeface(out GlyphTypeface? glyphTypeface))
                {
                    var text = "1";
                    var fontSize = 30;

                    var formattedText = new FormattedText(text, CultureInfo.CurrentCulture,
                        System.Windows.FlowDirection.LeftToRight, typeface, fontSize, Brushes.Black, pixelsPerDip);
                    var sameGlyphTypefaceAndFormattedText = Math.Abs(formattedText.Baseline - glyphTypeface.Baseline * fontSize) < 0.01; 
                    var sameFontFamilyAndFormattedText = Math.Abs(formattedText.Baseline - fontFamily.Baseline * fontSize) < 0.01;

                    Debug.WriteLine($"""
                                     字体名： {name} - {typefaceName}
                                     斜体： {glyphTypeface.Style}
                                     加粗： {glyphTypeface.Weight}
                                     拉伸： {glyphTypeface.Stretch}
                                     基线 FontFamily： {fontFamily.Baseline}
                                     基线 GlyphTypeface： {glyphTypeface.Baseline}
                                     基线 FormattedText： {formattedText.Baseline / fontSize}
                                     基线相同 FontFamily == GlyphTypeface： {fontFamily.Baseline == glyphTypeface.Baseline}
                                     基线相近 GlyphTypeface ~ FormattedText： {sameGlyphTypefaceAndFormattedText}
                                     基线相近 FontFamily ~ FormattedText： {sameFontFamilyAndFormattedText}
                                     
                                     """);
                }
            }
        }
```

输出内容如下，也欢迎大家在自己的设备上运行以上代码

```
字体名： 更纱终端书呆黑体-简 - Regular
斜体： Normal
加粗： Normal
拉伸： Normal
基线 FontFamily： 0.965
基线 GlyphTypeface： 0.965
基线 FormattedText： 0.9650000000000001
基线相同 FontFamily == GlyphTypeface： True
基线相近 GlyphTypeface ~ FormattedText： True
基线相近 FontFamily ~ FormattedText： True

字体名： 汉仪南宫体简 - Regular
斜体： Normal
加粗： Normal
拉伸： Normal
基线 FontFamily： 0.998046875
基线 GlyphTypeface： 0.859375
基线 FormattedText： 0.998
基线相同 FontFamily == GlyphTypeface： False
基线相近 GlyphTypeface ~ FormattedText： False
基线相近 FontFamily ~ FormattedText： True

字体名： 汉仪南宫体简 - Oblique
斜体： Oblique
加粗： Normal
拉伸： Normal
基线 FontFamily： 0.998046875
基线 GlyphTypeface： 0.859375
基线 FormattedText： 0.998
基线相同 FontFamily == GlyphTypeface： False
基线相近 GlyphTypeface ~ FormattedText： False
基线相近 FontFamily ~ FormattedText： True

字体名： 汉仪南宫体简 - Bold
斜体： Normal
加粗： Bold
拉伸： Normal
基线 FontFamily： 0.998046875
基线 GlyphTypeface： 0.859375
基线 FormattedText： 0.998
基线相同 FontFamily == GlyphTypeface： False
基线相近 GlyphTypeface ~ FormattedText： False
基线相近 FontFamily ~ FormattedText： True

字体名： 方正硬笔行书简体_非压缩版 - Regular
斜体： Normal
加粗： Normal
拉伸： Normal
基线 FontFamily： 0.82421875
基线 GlyphTypeface： 0.76953125
基线 FormattedText： 0.8242222222222222
基线相同 FontFamily == GlyphTypeface： False
基线相近 GlyphTypeface ~ FormattedText： False
基线相近 FontFamily ~ FormattedText： True

...
```
