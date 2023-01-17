# WPF 简单聊聊如何使用 DrawGlyphRun 绘制文本

在 WPF 里面，提供的使用底层的方法绘制文本是通过 DrawGlyphRun 的方式，此方法适合用在需要对文本进行精细控制的定制化控件上。此方法特别底层而让调用方法比较复杂，本文告诉大家一些简单的使用方法

<!--more-->
<!-- CreateTime:2021/9/29 19:09:47 -->

<!-- 发布 -->

本文也属于 WPF 渲染系列博客，更多渲染相关博客请看 [渲染相关](https://blog.lindexi.com/post/%E6%B8%B2%E6%9F%93 )

在开始之前，我是来劝退的，如果没有特别的需求，还是不推荐使用 DrawGlyphRun 的方式进行文本绘制。本文不会告诉大家特别基础的知识，基础部分还请看官方文档： [GlyphRun Class (System.Windows.Media)](https://docs.microsoft.com/en-us/dotnet/api/system.windows.media.glyphrun?WT.mc_id=WD-MVP-5003260 )

如果可以的话，顺便也将 [DirectWrite](https://docs.microsoft.com/en-us/windows/win32/directwrite/introducing-directwrite?WT.mc_id=WD-MVP-5003260 ) 的[官方文档](https://docs.microsoft.com/en-us/windows/win32/directwrite/introducing-directwrite?WT.mc_id=WD-MVP-5003260)也读一次

如果期望了解 WPF 底层是如何将 GlyphRun 渲染，请参阅 [dotnet 读 WPF 源代码笔记 渲染层是如何将字符 GlyphRun 画出来的](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-%E6%B8%B2%E6%9F%93%E5%B1%82%E6%98%AF%E5%A6%82%E4%BD%95%E5%B0%86%E5%AD%97%E7%AC%A6-GlyphRun-%E7%94%BB%E5%87%BA%E6%9D%A5%E7%9A%84.html )

使用 DrawGlyphRun 方法之前需要拿到一个 DrawingContext 对象，而在调用此方法时，重要的参数是 GlyphRun 对象，此对象包含了大量的参数，本文将来告诉大家这些的参数的用法

## 例子

新建一个空 WPF 项目用来做例子

在 MainWindow 的 Loaded 事件里面，创建 DrawingVisual 用来获取 DrawingContext 对象

```csharp
        public MainWindow()
        {
            InitializeComponent();

            Loaded += MainWindow_Loaded;
        }

        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            var drawingVisual = new DrawingVisual();
            using (var drawingContext = drawingVisual.RenderOpen())
            {

            }
            Background = new VisualBrush(drawingVisual);
        }
```

默认作为 Background 的 Brush 将会被撑开，为了让后续绘制的文本有指定的尺寸，绘制一个和窗口相同大小的矩形，这样就可以让 `drawingVisual.Drawing.Bounds` 的尺寸和窗口相同

```csharp
using (var drawingContext = drawingVisual.RenderOpen())
{
    drawingContext.DrawRectangle(Brushes.Black, null, new Rect(0, 0, ActualWidth, ActualHeight));
}
```

## 准备

在使用 DrawGlyphRun 绘制需要创建 GlyphRun 对象，需要有以下参数才能构建出绘制的文本内容

- 字体
- 字号
- 文本内容
- 文本绘制画刷
- 文本绘制的坐标

尽管 GlyphRun 对象需要的参数很多，然而很多参数都是可以默认获取的

## 字体

在 GlyphRun 里面需要的字体不是 FontFamily 而是需要传入的是 GlyphTypeface 对象。好在 GlyphTypeface 对象就是可以从 FontFamily 获取的

每个字体都相当于有一族，多个 Typeface 对象，如下面代码可以获取第一个 Typeface 对象

```csharp
var fontFamily = new FontFamily("微软雅黑");
Typeface typeface = fontFamily.GetTypefaces().First();
```

如果此字体是成功安装的，清真的字体，那么可以通过如下代码获取到 GlyphTypeface 对象

```csharp
bool success = typeface.TryGetGlyphTypeface(out GlyphTypeface glyphTypeface);
```

大部分字体都能成功拿到，如果不能成功那么，那么就需要自己走字体 Fallback 换个字体啦，或者炸掉。自己决定如果给定的字体创建失败了，则使用什么字体代替的方法叫做字体 Fallback 算法

关于如何做字体的回滚策略，还请参阅下文 字体回滚策略 内容

## 文字编号

每个文字在字体里面都可以有自己的编号，需要通过 CharacterToGlyphMap 获取对应的值

```csharp
var text = "林德熙abc123ATdVACC";

List<ushort> glyphIndices = new List<ushort>();

for (var i = 0; i < text.Length; i++)
{
    var c = text[i];
    var glyphIndex = glyphTypeface.CharacterToGlyphMap[c];
    glyphIndices.Add(glyphIndex);
}
```

需要同时在 GlyphRun 传入编号和 Unicode 的值

## 设置字号

在 GlyphRun 里面，支持输入多个文字和单个文字，在输入时，可以给每个文字指定字号。字号其实是一个上层的概念，而在 GlyphRun 需要使用底层的文本渲染概念，也就是字符的 AdvanceWidth 的值。简单的获取 AdvanceWidth 的方法如下

```csharp
List<double> advanceWidths = new List<double>();

for (var i = 0; i < text.Length; i++)
{
    var c = text[i];

    var width = glyphTypeface.AdvanceWidths[glyphIndex] * fontSize;
    advanceWidths.Add(width);
}
```

以上代码将字符串每个文字都设置相同的字号，但是大家可以根据需求，给每个文字都设置字号。对于等宽字符来说，每个字符的 AdvanceWidths 对应的值都应该是相同的。对于非等宽字符，可以在特殊排版需求的时候，强行设置为等宽的值

字符都是等比的，因此只需要设置宽度即可，设置字宽等于设置字号

## 设置字体偏移

在 GlyphRun 的高级用法里面，是允许设置文字的偏移量。文字的偏移量是一个文字的排版的基础值，推荐大家写一点代码去摸索一下他的规则

```csharp
List<Point> glyphOffsets = new List<Point>();
var fontSize = 30;

for (var i = 0; i < text.Length; i++)
{
    var c = text[i];

    // 只是决定每个字的偏移量，记得加上 i 乘以哦。字符最好是叠加上 fontSize 的值，使用 fontSize 的倍数
    glyphOffsets.Add(new Point(fontSize * i, 0));
}
```

在 GlyphRun 里面，文字的偏移量非必须的，可以传入为空值，因此以上代码是非必须的，只有需要控制每个字的偏移量的时候才需要用到。此偏移量不是相对坐标值，只是偏移量而已，相对来说比较绕

## 文本偏移

在 DrawGlyphRun 方法里面是不包含文本的坐标的参数的，需要在 GlyphRun 对象里面设置整个文本的起始坐标，如下面代码准备好文本的 X 和 Y 坐标值

```csharp
    var location = new Point(10, 100);
```

上面代码只是例子而已，还请替换为你的业务代码的需要绘制的文本坐标

但是需要知道的是在 GlyphRun 里面传入的是 BaseLine 而不是 Location 的值，相互转换的逻辑需要根据 FontFamily 的 Baseline 的值才能计算，代码如下

```csharp
        /// <summary>
        /// 获取指定字体的baseline
        /// </summary>
        /// <param name="fontFamily"></param>
        /// <param name="fontRenderingEmSize"></param>
        /// <returns></returns>
        public static double GetBaseline(this FontFamily fontFamily, double fontRenderingEmSize)
        {
            var baseline = fontFamily.Baseline;

            var renderingEmSize = fontRenderingEmSize;

            var value = baseline * renderingEmSize;
            return value;
        }

        location = new Point(location.X, location.Y + fontFamily.GetBaseline(fontSize));
```

以上代码是将 GetBaseline 的返回值给到 location 的 Y 值，这适合用在水平布局文本上。如果是垂直排版的文本，自然就需要放在水平方向。请根据你的业务代码修改以上逻辑

## 语言文化

如果需要支持特殊的文本内容，就需要设置特别的语言文化，默认使用 IetfLanguageTag 即可

```csharp
                XmlLanguage defaultXmlLanguage =
                    XmlLanguage.GetLanguage(CultureInfo.CurrentUICulture.IetfLanguageTag);
```

## DPI

在新的 GlyphRun 的构造里面要求传入 DPI 的值用于清晰化显示，在旧版本的，如 .NET Framework 4.5 版本是不需要的

官方推荐的获取 DPI 的方法是根据当前文本将要渲染出来的控件获取控件的 DPI 的值，通过此方法可以支持多屏幕不同 DPI 的感知。本文提供的方法是获取主窗口，因为本文的例子是在主窗口绘制文本

```csharp
    var pixelsPerDip = (float) VisualTreeHelper.GetDpi(Application.Current.MainWindow).PixelsPerDip;
```

## 绘制文本

在准备完成之后，即可创建 GlyphRun 用来绘制

```csharp
  var glyphRun = new GlyphRun
  (
      glyphTypeface,
      bidiLevel: 0,
      isSideways: false,
      renderingEmSize: fontSize,
      pixelsPerDip: pixelsPerDip,   // 只有在高版本的 .NET 才有此参数
      glyphIndices: glyphIndices,
      baselineOrigin: location,     // 设置文本的偏移量
      advanceWidths: advanceWidths, // 设置每个字符的字宽，也就是字号
      glyphOffsets: null,           // 设置每个字符的偏移量，可以为空
      characters: text.ToCharArray(),
      deviceFontName: null,
      clusterMap: null,
      caretStops: null,
      language: defaultXmlLanguage
  );

  drawingContext.DrawGlyphRun(Brushes.White, glyphRun);
```

请将 Brushes.White 替换为字体前景色的画刷

以上的 `deviceFontName` 参数留空即可，这是一个没有什么作用的参数，详细请看 [dotnet 读 WPF 源代码笔记 GlyphRun 的 DeviceFontName 的功能是什么](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-GlyphRun-%E7%9A%84-DeviceFontName-%E7%9A%84%E5%8A%9F%E8%83%BD%E6%98%AF%E4%BB%80%E4%B9%88.html )

以上即可完成文本的绘制，这是一个底层的方式，看起来也很简单

## 创建成本

创建一个 GlyphRun 对象的成本有多高？是否需要申请很多资源？其实创建时仅仅只是创建了一个 CLR 对象而已，里面也只有很多的字段，成本非常低。在创建时不会用到任何非托管的资源，只是一个对象而已

只有在被绘制的时候，才会申请 DirectWrite 的相关资源

## 获取几何对象

通过 BuildGeometry 方法可以从 GlyphRun 对象创建几何对象，如下面代码

```csharp
var geometry = glyphRun.BuildGeometry();
```

获取几何对象可以用此几何对象做特殊的逻辑，如文字描边等

需要小心的是调用 BuildGeometry 方法是有一定成本的，底层将需要从文本渲染为 Geometry 对象，中间需要经过 MIL 层。建议是能复用就复用，而不要每次都创建

但是在复用时，需要了解的是，不同的字号，创建出来的 Geometry 对象，不一定是相同的，这是为了清晰化显示的考虑。如字体比较小的时候，将会删减一些笔画等

## 获取文本的渲染尺寸

可以通过如下代码获取文本的渲染尺寸，也可以通过如下方法获取单个字符的渲染尺寸

```csharp
  var computeInkBoundingBox = glyphRun.ComputeInkBoundingBox();
  var matrix = new Matrix();
  matrix.Translate(location.X, location.Y);
  computeInkBoundingBox.Transform(matrix);
  //相对于run.BuildGeometry().Bounds方法，run.ComputeInkBoundingBox()会多出一个厚度为1的框框，所以要减去
  if (computeInkBoundingBox.Width >= 2 && computeInkBoundingBox.Height >= 2)
  {
      computeInkBoundingBox.Inflate(-1, -1);
  }
```

以上的 computeInkBoundingBox 就是文本的绘制的尺寸，相对的坐标是文本的左上角，因此需要通过 location 叠加变换才能让此矩形和文本渲染重叠

```csharp
     drawingContext.DrawRectangle(Brushes.Blue, null, computeInkBoundingBox);
```

文本的渲染尺寸也就是文本的字墨尺寸，此概念是文本排版概念

## 获取文本的文字布局尺寸

可以通过以上代码的 width 获取文本的字面的布局宽度，而布局高度则需要根据 BaseLine 等属性获取，代码如下

```csharp
        /// <summary>
        /// 获取<see cref="GlyphRun"/>的Size
        /// </summary>
        /// <param name="run"></param>
        /// <param name="lineSpacing"></param>
        /// <returns></returns>
        public static Size GetSize(this GlyphRun run, double lineSpacing)
        {
            var renderingEmSize = run.FontRenderingEmSize;
            var height = lineSpacing * renderingEmSize;
            double width = 0;
            foreach (var index in run.GlyphIndices)
            {
                width += run.GlyphTypeface.AdvanceWidths[index];
            }

            width = width * renderingEmSize;
            return new Size(width, height);
        }
```

调用方法是 `var glyphSize = glyphRun.GetSize(fontFamily.LineSpacing);` 即可拿到文字的布局尺寸

## 字体回滚策略

字体的回滚策略可以比较佛系，毕竟是找不到字体了，此时就是从已安装的字体找到一个还能用的字体代替上去

在 WPF 源代码里面，可以看到底层的 Fallback 字体是 `#GLOBAL USER INTERFACE` 这个特殊的字体，为了保持和 TextBlock 差不多的逻辑，可以使用如下方法作为字体回滚

```csharp
    /// <summary>
    /// 用于回滚的字体对象<see cref="FontFamily"/>
    /// </summary>
    public class FallBackFontFamily
    {
        private const string FallBackFontFamilyName = "#GLOBAL USER INTERFACE";
        private FontFamily FallBack { get; } = new FontFamily(FallBackFontFamilyName);

        private FallBackFontFamily(CultureInfo culture)
        {
            FontFamilyItems = FallBack.FamilyMaps
                .Where(map => map.Language == null || map.Language.MatchCulture(culture))
                .Select(map => new FontFamilyMapItem(map)).ToList();
        }

        private IEnumerable<FontFamilyMapItem> FontFamilyItems { get; }

        /// <summary>
        /// 获取<see cref="FallBackFontFamily"/>对象的单例
        /// </summary>
        public static FallBackFontFamily Instance => FallBackFontFamilyLazy.Value;

        private static readonly Lazy<FallBackFontFamily> FallBackFontFamilyLazy =
            new Lazy<FallBackFontFamily>(() => new FallBackFontFamily(CultureInfo.CurrentCulture));

        /// <summary>
        /// 尝试获取fallback的字体名称
        /// </summary>
        /// <param name="unicodeChar"></param>
        /// <param name="familyName"></param>
        /// <returns></returns>
        public bool TryGetFallBackFontFamily(char unicodeChar, out string familyName)
        {
            var mapItem = FontFamilyItems.FirstOrDefault(item => item.InRange(unicodeChar));
            familyName = null;

            if (mapItem !=null)
            {
                familyName = mapItem.Target;
                return true;
            }
            return false;
        }
    }
```

以上字体也就是 FontFamily.FontFamilyGlobalUI 属性的值，请看以下的 WPF 框架源代码

```csharp
        internal const string GlobalUI = "#GLOBAL USER INTERFACE";

        internal static FontFamily FontFamilyGlobalUI = new FontFamily(GlobalUI);
```

默认在 WPF 的 Typeface 创建就包含了此逻辑，请看 Typeface 的源代码

```csharp
        public Typeface(
            FontFamily      fontFamily,
            FontStyle       style,
            FontWeight      weight,
            FontStretch     stretch
            )
            : this(
                fontFamily,
                style,
                weight,
                stretch,
                FontFamily.FontFamilyGlobalUI
                )
        {}
```

因此以上的回滚代码的意义其实不大，不过可以通过以上代码添加自己期望的字体回滚列表，如自己在应用程序里面带了特殊的字体，期望在找不到字体的时候使用自己的字体，就可以使用上面提供的回滚策略代码，使用方法如下

```csharp
            if (typeface.TryGetGlyphTypeface(out var glyph))
            {
                // 忽略代码
            }
            else if (FallBackFontFamily.Instance.TryGetFallBackFontFamily(unicodeChar, out var familyName))
            {
            	// 上面代码的 unicodeChar 就是传入的文本的字符
            	// 通过上面代码可以拿到回滚的字体是否包含此字符的定义
            }
            else
            {
                // 没有可以支持此字符的字体，那就看业务逻辑的处理啦
            }
```

## 代码

### 例子

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/581ea123df0d1067ec1ed3527e8b85edb2fd082e/NiwejabainelFehargaye) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/581ea123df0d1067ec1ed3527e8b85edb2fd082e/NiwejabainelFehargaye) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 581ea123df0d1067ec1ed3527e8b85edb2fd082e
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 NiwejabainelFehargaye 文件夹

### 轻文本

实现一个和 TextBox 差很多的单行轻文本最简代码如下

```csharp
    class Foo : UIElement
    {
        public string Text { set; get; } = string.Empty;

        protected override void OnRender(DrawingContext drawingContext)
        {
            var fontFamily = new FontFamily("微软雅黑");

            var fontSize = 15;
            var y = 0;
            drawingContext.PushOpacity(0.3);
            foreach (var typeface in fontFamily.GetTypefaces().Skip(1).Take(1))
            {
                double offset = 3;

                var baseLine = fontFamily.GetBaseline(fontSize);

                if (typeface.TryGetGlyphTypeface(out var glyphTypeface))
                {
                    foreach (var c in Text)
                    {
                        if (glyphTypeface.CharacterToGlyphMap.TryGetValue(c, out var glyphIndex))
                        {
                            // 在排版，不适合将每个字符的宽度独立进行计算。有很多字符是需要重叠布局的
                            var width = glyphTypeface.AdvanceWidths[glyphIndex] * fontSize;
                            width = GlyphExtension.RefineValue(width);

#pragma warning disable 618 // 忽略调用废弃构造函数
                            var glyphRun = new GlyphRun(
#pragma warning restore 618
                                glyphTypeface,
                                0,
                                false,
                                fontSize,
                                new[] { glyphIndex },
                                new Point(offset, baseLine + y),
                                new[] { width },
                                DefaultGlyphOffsetArray,
                                new char[] { c },
                                null,
                                null,
                                null, DefaultXmlLanguage);

                            drawingContext.DrawLine(new Pen(Brushes.Black, 2), new Point(offset, y), new Point(offset + width, y));

                            drawingContext.DrawGlyphRun(Brushes.Coral, glyphRun);

                            var glyphSize = glyphRun.GetSize(fontFamily.LineSpacing);

                            drawingContext.DrawRectangle(null, new Pen(Brushes.Black, 2), new Rect(new Point(offset, y), glyphSize));

                            // 布局的字符宽度
                            offset += width;
                        }
                    }
                }

                y += fontSize;
            }
            drawingContext.Pop();
        }

        private static readonly Point[] DefaultGlyphOffsetArray = new Point[] { new Point() };

        private static readonly XmlLanguage DefaultXmlLanguage =
            XmlLanguage.GetLanguage(CultureInfo.CurrentUICulture.IetfLanguageTag);
    }
```

以上代码只是单个字符进行绘制，用于了解每个字符对应的布局值，也就是如上的 DrawRectangle 绘制的内容

上面代码的 GetBaseline 等都是辅助方法，可以从本文上面找到代码，也可以通过如下方式获取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin fe704afdd32edb05005b1f35bcc87dc59c900040
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 NiwejabainelFehargaye 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
