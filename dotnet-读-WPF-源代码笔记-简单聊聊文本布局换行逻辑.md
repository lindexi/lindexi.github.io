
# dotnet 读 WPF 源代码笔记 简单聊聊文本布局换行逻辑

在 WPF 里面，带了基础的文本库功能，如 TextBlock 等。文本库排版的重点是在文本的分行逻辑，也就是换行逻辑，如何计算当前的文本字符串到达哪个字符就需要换到下一行的逻辑就是文本布局的重点模块。本文来简单聊聊 WPF 的文本布局逻辑

<!--more-->


<!-- CreateTime:2021/10/18 8:31:15 -->


<!-- 标签：WPF，渲染，WPF源代码 -->
<!-- 发布 -->

先写给不想阅读细节的大佬们了解 WPF 文本模块的布局逻辑： 文本的排版和渲染是分开的两个模块。 文本逻辑在排版里面，核心都会调用到 TextFormatterImp 里面，在这里将会通过 SimpleTextLine 尝试进行布局排版，在 SimpleTextLine 里面将会判断当前的文本字符串是否刚好一行能放下，如果可以放下，那么就使用当行方式显示。这是最为简单的，实现逻辑就是通过 Typeface 的 GlyphMetrics 的 AdvanceWidth 列表获取每个字符的排版宽度，将排版宽度乘以渲染字号即可获取每个字符占用的渲染布局宽度，将所有字符的占用布局框架之和 与可用行宽度进行比较，如果小于行宽度则进行单行布局

如果超过单行布局的能力，则进入 TextMetrics 的 FullTextLine 方法。此方法将使用到没有开源的 PresentationNative.dll 提供的 LoCreateLine 方法进行文本排版逻辑。在 PresentationNative 里面将会调用系统多语言处理 （也许是叫 TFS 但如果叫错了还请大佬们教教我）进行文本的复杂排版行为，包括进行合写字如蒙文藏文的排版逻辑。这部分复杂排版是需要系统层多语言的支持的，包含了复杂的语言文化规则

下面就是细节部分的逻辑

在 TextBlock 等的底层也是用到了 TextFormatterImp 的文本排版功能进行排版，然后进行渲染。渲染部分本文就不聊了

如在 TextBlock 的 OnRender 或 MeasureOverride 方法里面，都会调用 CreateLine 方法创建 Line 对象，接着通过 Line 对象的 Format 方法层层调用到 TextFormatterImp 里面，大概代码如下

```csharp
    [ContentProperty("Inlines")]
    [Localizability(LocalizationCategory.Text)]
    public class TextBlock : FrameworkElement, IContentHost, IAddChildInternal, IServiceProvider
    {
        protected sealed override Size MeasureOverride(Size constraint)
        {
        	// 忽略逻辑
                // Create and format lines until end of paragraph is reached.
                // Since we are disposing line object, it can be reused to format following lines.
                Line line = CreateLine(lineProperties);
                while (!endOfParagraph)
                {
                    using(line)
                    {
                        // Format line. Set showParagraphEllipsis flag to false because we do not know whether or not the line will have
                        // paragraph ellipsis at this time. Since TextBlock is auto-sized we do not know the RenderSize until we finish Measure
                        line.Format(dcp, contentSize.Width, GetLineProperties(dcp == 0, lineProperties), textLineBreakIn, _textBlockCache._textRunCache, /*Show paragraph ellipsis*/ false);

                        // 忽略其他逻辑
                    }
                }
        }
    }

    // ----------------------------------------------------------------------
    // Text line formatter.
    // ----------------------------------------------------------------------
    internal abstract class Line : TextSource, IDisposable
    {
        // ------------------------------------------------------------------
        // Create and format text line.
        //
        //      lineStartIndex - index of the first character in the line
        //      width - wrapping width of the line
        //      lineProperties - properties of the line
        //      textRunCache - run cache used by text formatter
        //      showParagraphEllipsis - true if paragraph ellipsis is shown 
        //                              at the end of the line
        // ------------------------------------------------------------------
        internal void Format(int dcp, double width, TextParagraphProperties lineProperties, TextLineBreak textLineBreak, TextRunCache textRunCache, bool showParagraphEllipsis)
        {
        	// 忽略代码
            _line = _owner.TextFormatter.FormatLine(this, dcp, width, lineProperties, textLineBreak, textRunCache);
        }
    }

    internal sealed class TextFormatterImp : TextFormatter
    {
        public override TextLine FormatLine(
            TextSource                  textSource,
            int                         firstCharIndex,
            double                      paragraphWidth,
            TextParagraphProperties     paragraphProperties,
            TextLineBreak               previousLineBreak,
            TextRunCache                textRunCache
            )
        {
            return FormatLineInternal(
                textSource,
                firstCharIndex,
                0,   // lineLength
                paragraphWidth,
                paragraphProperties,
                previousLineBreak,
                textRunCache
                );
        }

        /// <summary>
        /// Format and produce a text line either with or without previously known
        /// line break point.
        /// </summary>
        private TextLine FormatLineInternal(
            TextSource                  textSource,
            int                         firstCharIndex,
            int                         lineLength,
            double                      paragraphWidth,
            TextParagraphProperties     paragraphProperties,
            TextLineBreak               previousLineBreak,
            TextRunCache                textRunCache
            )
        {
        	// 忽略代码
        }
    }
```

通过上面代码可以看到在 WPF 框架，核心的文本排版逻辑是在 FormatLineInternal 方法里面

在 FormatLineInternal 里面将会先使用 SimpleTextLine 尝试作为一行进行布局，假设文本一行能放下，也就不需要复杂的排版逻辑，可以提升很大的性能。如果一行放不下，那就通过 TextMetrics 的 FullTextLine 进行复杂的排版逻辑

```csharp
        /// <summary>
        /// Format and produce a text line either with or without previously known
        /// line break point.
        /// </summary>
        private TextLine FormatLineInternal(
            TextSource                  textSource,
            int                         firstCharIndex,
            int                         lineLength,
            double                      paragraphWidth,
            TextParagraphProperties     paragraphProperties,
            TextLineBreak               previousLineBreak,
            TextRunCache                textRunCache
            )
        {
            // prepare formatting settings
            FormatSettings settings = PrepareFormatSettings(/*忽略传入参数*/);

            TextLine textLine = null;

            if ( /*可以进行单行排版的文本*/ )
            {
                // simple text line.
                textLine = SimpleTextLine.Create(/*忽略传入参数*/);
            }

            if (textLine == null)
            {
                // content is complex, creating complex line
                textLine = new TextMetrics.FullTextLine(/*忽略传入参数*/);
            }

            return textLine;
        }
```

在文本进行复杂排版，就需要用到没有开源的 PresentationNative.dll 提供的和系统层的多语言对接的功能。本文就仅来了解 SimpleTextLine 的实现

在 SimpleTextLine 里面，实现的逻辑是将当前的文本在传入的宽度内进行一行布局，如果能在一行进行布局，那就返回值，否则返回空

文本里面有段落和行和 TextRun 的三个概念，在开始了解 WPF 的代码之前，咱先定义这三个不同的概念。一个文本里面包含有多段，默认采用换行符作为分段。也就是说在一段里面是不会存在多个换行符的。一个段落里面将会因为文本框的宽度限制而存在多行。一行文本里面，将会因为文本属性的不同将文本分为多个 TextRun 对象

也就是最简单的文本就是一个字符，一个字符是一个 TextRun 放在一行里面，这一行放在一段里面

在 SimpleTextLine 的 Create 方法将层层调用进入到 CreateSimpleTextRun 方法里面，也就是说在一行里面将会一个个 TextRun 进行创建，创建的时候同时判断当前的文本剩余宽度是否足够

在 CreateSimpleTextRun 方法里面将会调用 Typeface.CheckFastPathNominalGlyphs 方法进行快速的创建，这个方法是没有开放出来给开发者使用的，调用这个方法可以绕过很多判断逻辑，性能很高

在 CheckFastPathNominalGlyphs 方法里面，将会使用 Typeface 的 TypefaceMetrics 属性作为 GlyphTypeface 类型的对象。此对象依然可以使用到没有开放给开发者使用的 GetGlyphMetricsOptimized 方法。如方法命名可以看到，这是一个有很多性能优化的方法。此方法将拿到文本字符串对应的 glyphIndices 和 glyphMetrics 两个数组，分别表示的是字符对应在 Glyph 的序号以及 Glyph 的信息，代码如下

```csharp
            ushort[] glyphIndices = BufferCache.GetUShorts(charBufferRange.Length);
            MS.Internal.Text.TextInterface.GlyphMetrics[] glyphMetrics = ignoreWidths ? null : BufferCache.GetGlyphMetrics(charBufferRange.Length);

            glyphTypeface.GetGlyphMetricsOptimized(charBufferRange, 
                                                   emSize,
                                                   pixelsPerDip,
                                                   glyphIndices,
                                                   glyphMetrics,
                                                   textFormattingMode,
                                                   isSideways
                                                   );
```

以上的 `glyphIndices` 变量和 `glyphMetrics` 都是从 BufferCache 获取的，大部分排版逻辑都需要额外申请内存。此方法对比开放给开发者使用的版本的优势在于可以批量获取，给开发者使用的版本只能一个个字符获取，性能上远远不如调用此方法获取。更多关于开发者使用文本排版，请看 [WPF 简单聊聊如何使用 DrawGlyphRun 绘制文本](https://blog.lindexi.com/post/WPF-%E7%AE%80%E5%8D%95%E8%81%8A%E8%81%8A%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-DrawGlyphRun-%E7%BB%98%E5%88%B6%E6%96%87%E6%9C%AC.html)

在拿到以上两个变量之后，即可进行计算每个字符的排版宽度，此计算方法将会让计算出来的值和实际渲染尺寸有一些误差。然而此排版方法只是计算是否在一行里面足够放下文本，有一些误差不会影响到结果。因为如果能一行进行排版，那就走以上的方法，是高性能模式。如果一行不能排版，那就通过系统层的语言文化进行排版，可以符合业务的需求

大概的计算逻辑如下

```csharp
            //
            // This block will advance until one of:
            // 1. The end of the charBufferRange is reached
            // 2. The charFlags have some of the charFlagsMask values
            // 3. Glyph index is 0 (unless symbol font)
            // 4. totalWidth > widthMax
            //
            
            while(
                    i < charBufferRange.Length // charBufferRange 就是文本的 Char 列表
                &&  (ignoreWidths || totalWidth <= widthMax) // totalWidth 是当前文本已排版的字符的宽度之和
                &&  ((charFlags & charFlagsMask) == 0)
                &&  (glyph != 0 || symbolTypeface) // 在 glyph 是 0 时，表示的是当前没有字符，相当于 \0 字符。但是符号字体不在此范围
                )
            {
                char ch = charBufferRange[i++];
                if (ch == TextStore.CharLineFeed || ch == TextStore.CharCarriageReturn || (breakOnTabs && ch == TextStore.CharTab))
                {
                    --i;
                    break;
                }
                else
                {
                    int charClass = (int)Classification.GetUnicodeClassUTF16(ch);
                    charFlags = Classification.CharAttributeOf(charClass).Flags;
                    charFastTextCheck &= charFlags;

                    glyph = glyphIndices[i-1];
                    if (!ignoreWidths)
                    {
                        totalWidth += TextFormatterImp.RoundDip(glyphMetrics[i - 1].AdvanceWidth * designToEm, pixelsPerDip, textFormattingMode) * scalingFactor;
                    }
                }
            }
```

上面逻辑核心就是 `totalWidth <= widthMax` 判断，判断当前布局的字符宽度之和是否小于可以使用的宽度。如果大于那就表示这一行放不下此字符串

计算单个字符占用的宽度使用的是 `glyphMetrics[i - 1].AdvanceWidth * designToEm` 进行计算，而 RoundDip 只是加上 Dpi 的辅助计算而已。以上的 AdvanceWidth 将是字符的宽度比例，可以乘以 designToEm 设计时的字号计算出 WPF 单位的宽度

也就是文本的单行排版里面就是通过各个字符的设计时宽度计算是否可以在一行排列，如果可以那就采用此优化，不再进行复杂文本排版，进入渲染逻辑

更多渲染相关博客请看 [渲染相关](https://blog.lindexi.com/post/%E6%B8%B2%E6%9F%93 )

在 WPF 框架，一开始设计是不存在 DriectWrite 的，是在 .NET Framework 3.5 之后才加入的，在 .NET Framework 4.0 发布，这里是一个[官方的引用](https://github.com/dotnet/wpf/issues/5509#issuecomment-946861616) 也在 WPF 编程宝典有讲到





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。