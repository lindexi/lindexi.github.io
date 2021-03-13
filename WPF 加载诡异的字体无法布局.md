# WPF 加载诡异的字体无法布局

如果在系统里面存在诡异的字体，同时自己的 WPF 中有一个控件尝试使用这个字体放在界面中，那么将会在界面布局过程炸了，整个控件或者整个界面布局都无法继续

<!--more-->
<!-- 发布 -->

本文本来是由[吕水大大](http://blog.walterlv.com/)发布的，但是他没空写，于是我就成为了写博客的工具人

有一个用户报告了软件在他的电脑上打不开列出本机字体列表，于是[吕水大大](http://blog.walterlv.com/)就去远程他的设备，在用户的设备上找到了一个诡异的字体，加载这个字体的时候，将会在 `MS.Internal.Text.TextInterface.Font.CreateFontFace` 抛出异常。而且有趣的是 Win10 的 UWP 版的设置里面是找不到这个字体的，原因是 UWP 版本也会在读取此字体的时候炸了

复现的步骤如下，先从 [https://github.com/walterlv/Walterlv.Demo.WpfBadFontCrash](https://github.com/walterlv/Walterlv.Demo.WpfBadFontCrash) 下载代码，在代码仓库里面可以找到 不给糖就捣蛋的万圣节.TTF 这个字体，值得一说的是，诡异的字体有很多，这个字体只是一个例子。双击安装一下这个字体到你的系统上

然后新建一个 WPF 或 UWP 程序，在界面里面添加一个 TextBlock 然后在 TextBlock 里面采用此字体，如下面代码

```xml
    <TextBlock Text="ABCDEFGH" FontFamily="不给糖就捣蛋的万圣节" />
```

参与运行此 WPF 程序，将会看到如下提示

```
System.IO.FileFormatException: Invalid file format.
   at MS.Internal.Text.TextInterface.Native.Util.ConvertHresultToException(Int32 hr)
   at MS.Internal.Text.TextInterface.Font.CreateFontFace()
   at MS.Internal.Text.TextInterface.Font.AddFontFaceToCache()
   at MS.Internal.Text.TextInterface.Font.GetFontFace()
   at System.Windows.Media.GlyphTypeface..ctor(Font font)
   at MS.Internal.FontFace.PhysicalFontFamily.GetGlyphTypeface(FontStyle style, FontWeight weight, FontStretch stretch)
   at MS.Internal.FontFace.PhysicalFontFamily.MS.Internal.FontFace.IFontFamily.GetTypefaceMetrics(FontStyle style, FontWeight weight, FontStretch stretch)
   at System.Windows.Media.Typeface.ConstructCachedTypeface()
   at System.Windows.Media.Typeface.get_CachedTypeface()
   at System.Windows.Media.Typeface.CheckFastPathNominalGlyphs(CharacterBufferRange charBufferRange, Double emSize, Single pixelsPerDip, Double scalingFactor, Double widthMax, Boolean keepAWord, Boolean numberSubstitution, CultureInfo cultureInfo, TextFormattingMode textFormattingMode, Boolean isSideways, Boolean breakOnTabs, Int32& stringLengthFit)
   at MS.Internal.TextFormatting.SimpleRun.CreateSimpleTextRun(CharacterBufferRange charBufferRange, TextRun textRun, TextFormatterImp formatter, Int32 widthLeft, Boolean emergencyWrap, Boolean breakOnTabs, Double pixelsPerDip)
   at MS.Internal.TextFormatting.SimpleRun.Create(FormatSettings settings, CharacterBufferRange charString, TextRun textRun, Int32 cp, Int32 cpFirst, Int32 runLength, Int32 widthLeft, Int32 idealRunOffsetUnRounded, Double pixelsPerDip)
   at MS.Internal.TextFormatting.SimpleTextLine.Create(FormatSettings settings, Int32 cpFirst, Int32 paragraphWidth, Double pixelsPerDip)
   at MS.Internal.TextFormatting.TextFormatterImp.FormatLineInternal(TextSource textSource, Int32 firstCharIndex, Int32 lineLength, Double paragraphWidth, TextParagraphProperties paragraphProperties, TextLineBreak previousLineBreak, TextRunCache textRunCache)
   at MS.Internal.TextFormatting.TextFormatterImp.FormatLine(TextSource textSource, Int32 firstCharIndex, Double paragraphWidth, TextParagraphProperties paragraphProperties, TextLineBreak previousLineBreak, TextRunCache textRunCache)
   at System.Windows.Controls.TextBlock.MeasureOverride(Size constraint)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at System.Windows.Controls.Grid.MeasureOverride(Size constraint)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at MS.Internal.Helper.MeasureElementWithSingleChild(UIElement element, Size constraint)
   at System.Windows.Controls.ContentPresenter.MeasureOverride(Size constraint)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at System.Windows.Controls.Decorator.MeasureOverride(Size constraint)
   at System.Windows.Documents.AdornerDecorator.MeasureOverride(Size constraint)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at System.Windows.Controls.Border.MeasureOverride(Size constraint)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at System.Windows.Window.MeasureOverrideHelper(Size constraint)
   at System.Windows.Window.MeasureOverride(Size availableSize)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at System.Windows.Interop.HwndSource.SetLayoutSize()
   at System.Windows.Interop.HwndSource.set_RootVisualInternal(Visual value)
   at System.Windows.Interop.HwndSource.set_RootVisual(Visual value)
   at System.Windows.Window.SetRootVisual()
   at System.Windows.Window.SetRootVisualAndUpdateSTC()
   at System.Windows.Window.SetupInitialState(Double requestedTop, Double requestedLeft, Double requestedWidth, Double requestedHeight)
   at System.Windows.Window.CreateSourceWindow(Boolean duringShow)
   at System.Windows.Window.CreateSourceWindowDuringShow()
   at System.Windows.Window.SafeCreateWindowDuringShow()
   at System.Windows.Window.ShowHelper(Object booleanBox)
   at System.Windows.Threading.ExceptionWrapper.InternalRealCall(Delegate callback, Object args, Int32 numArgs)
   at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(Object source, Delegate callback, Object args, Int32 numArgs, Delegate catchHandler)
```

如果上面的逻辑是放在 MainWindow 里面，那么意味着刚尝试创建窗口就凉凉了，可以看到窗口都创建失败。如果自己没有写 Dispatcher.UnhandledException 那么应用程序将会退出

代码请看 [https://github.com/walterlv/Walterlv.Demo.WpfBadFontCrash](https://github.com/walterlv/Walterlv.Demo.WpfBadFontCrash)

这个坑我报告给了 WPF 官方，请看 [WPF can not handle special damaged font · Issue #4283 · dotnet/wpf](https://github.com/dotnet/wpf/issues/4283)

对于 UWP 来说也一样，尝试在界面中放和上面 WPF 相同的代码，可以看到设计器给出了提示

![](http://image.acmx.xyz/lindexi%2F202131316131196.jpg)

运行 UWP 应用，将可以看到进入了下面代码

```csharp
#if DEBUG && !DISABLE_XAML_GENERATED_BREAK_ON_UNHANDLED_EXCEPTION
            UnhandledException += (sender, e) =>
            {
                if (global::System.Diagnostics.Debugger.IsAttached) global::System.Diagnostics.Debugger.Break();
            };
#endif
```

这里面的 e 的内容没有啥有用的信息，可以看到的代码如下

```
-		Exception	{"指示输入文件 (例如字体文件) 中的错误。\r\n\r\n指示输入文件 (例如字体文件) 中的错误。\r\n"}	System.Exception {System.Runtime.InteropServices.COMException}
```

也就说 UWP 在调用到更底层的时候炸掉了，其实也看不到堆栈。同时在 UWP 如果是在第一个界面中添加以上代码，那么即使在 App.xaml.cs 使用下面代码尝试接住，应用也是继续退出

```
        public App()
        {
            InitializeComponent();
            Suspending += OnSuspending;
            UnhandledException += App_UnhandledException;
        }

        private void App_UnhandledException(object sender, Windows.UI.Xaml.UnhandledExceptionEventArgs e)
        {
            e.Handled = true;
        }
```

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0fe2a367/KinubachekallHinuquba) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0fe2a367/KinubachekallHinuquba) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  


<!-- 

WPF can not handle special damaged font


* .NET Core Version: All
* Windows version: Any
* Does the bug reproduce also in WPF for .NET Framework 4.8?: Yes

 **Problem description:**

When we use the special damaged font, the WPF layout system will break. And we can not do something to handle it.

The step:

1. Clone the [https://github.com/walterlv/Walterlv.Demo.WpfBadFontCrash](https://github.com/walterlv/Walterlv.Demo.WpfBadFontCrash) repo
2. Install the 不给糖就捣蛋的万圣节.TTF font in the Walterlv.Demo.WpfBadFontCrash folder
3. Add the code to MainWindow.xaml and then build and run the application

    <TextBlock Text="ABCDEFGH" FontFamily="不给糖就捣蛋的万圣节" />
 
 **Actual behavior:** 


And you will find the application throw the exception.

System.IO.FileFormatException: Invalid file format.
   at MS.Internal.Text.TextInterface.Native.Util.ConvertHresultToException(Int32 hr)
   at MS.Internal.Text.TextInterface.Font.CreateFontFace()
   at MS.Internal.Text.TextInterface.Font.AddFontFaceToCache()
   at MS.Internal.Text.TextInterface.Font.GetFontFace()
   at System.Windows.Media.GlyphTypeface..ctor(Font font)
   at MS.Internal.FontFace.PhysicalFontFamily.GetGlyphTypeface(FontStyle style, FontWeight weight, FontStretch stretch)
   at MS.Internal.FontFace.PhysicalFontFamily.MS.Internal.FontFace.IFontFamily.GetTypefaceMetrics(FontStyle style, FontWeight weight, FontStretch stretch)
   at System.Windows.Media.Typeface.ConstructCachedTypeface()
   at System.Windows.Media.Typeface.get_CachedTypeface()
   at System.Windows.Media.Typeface.CheckFastPathNominalGlyphs(CharacterBufferRange charBufferRange, Double emSize, Single pixelsPerDip, Double scalingFactor, Double widthMax, Boolean keepAWord, Boolean numberSubstitution, CultureInfo cultureInfo, TextFormattingMode textFormattingMode, Boolean isSideways, Boolean breakOnTabs, Int32& stringLengthFit)
   at MS.Internal.TextFormatting.SimpleRun.CreateSimpleTextRun(CharacterBufferRange charBufferRange, TextRun textRun, TextFormatterImp formatter, Int32 widthLeft, Boolean emergencyWrap, Boolean breakOnTabs, Double pixelsPerDip)
   at MS.Internal.TextFormatting.SimpleRun.Create(FormatSettings settings, CharacterBufferRange charString, TextRun textRun, Int32 cp, Int32 cpFirst, Int32 runLength, Int32 widthLeft, Int32 idealRunOffsetUnRounded, Double pixelsPerDip)
   at MS.Internal.TextFormatting.SimpleTextLine.Create(FormatSettings settings, Int32 cpFirst, Int32 paragraphWidth, Double pixelsPerDip)
   at MS.Internal.TextFormatting.TextFormatterImp.FormatLineInternal(TextSource textSource, Int32 firstCharIndex, Int32 lineLength, Double paragraphWidth, TextParagraphProperties paragraphProperties, TextLineBreak previousLineBreak, TextRunCache textRunCache)
   at MS.Internal.TextFormatting.TextFormatterImp.FormatLine(TextSource textSource, Int32 firstCharIndex, Double paragraphWidth, TextParagraphProperties paragraphProperties, TextLineBreak previousLineBreak, TextRunCache textRunCache)
   at System.Windows.Controls.TextBlock.MeasureOverride(Size constraint)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at System.Windows.Controls.Grid.MeasureOverride(Size constraint)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at MS.Internal.Helper.MeasureElementWithSingleChild(UIElement element, Size constraint)
   at System.Windows.Controls.ContentPresenter.MeasureOverride(Size constraint)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at System.Windows.Controls.Decorator.MeasureOverride(Size constraint)
   at System.Windows.Documents.AdornerDecorator.MeasureOverride(Size constraint)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at System.Windows.Controls.Border.MeasureOverride(Size constraint)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at System.Windows.Window.MeasureOverrideHelper(Size constraint)
   at System.Windows.Window.MeasureOverride(Size availableSize)
   at System.Windows.FrameworkElement.MeasureCore(Size availableSize)
   at System.Windows.UIElement.Measure(Size availableSize)
   at System.Windows.Interop.HwndSource.SetLayoutSize()
   at System.Windows.Interop.HwndSource.set_RootVisualInternal(Visual value)
   at System.Windows.Interop.HwndSource.set_RootVisual(Visual value)
   at System.Windows.Window.SetRootVisual()
   at System.Windows.Window.SetRootVisualAndUpdateSTC()
   at System.Windows.Window.SetupInitialState(Double requestedTop, Double requestedLeft, Double requestedWidth, Double requestedHeight)
   at System.Windows.Window.CreateSourceWindow(Boolean duringShow)
   at System.Windows.Window.CreateSourceWindowDuringShow()
   at System.Windows.Window.SafeCreateWindowDuringShow()
   at System.Windows.Window.ShowHelper(Object booleanBox)
   at System.Windows.Threading.ExceptionWrapper.InternalRealCall(Delegate callback, Object args, Int32 numArgs)
   at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(Object source, Delegate callback, Object args, Int32 numArgs, Delegate catchHandler)

And the WPF layout system will break.

It also makes UWP application not work
 
 **Expected behavior:**

We can handle the exception to fix the WPF layout system
 
 **Minimal repro:**

[https://github.com/walterlv/Walterlv.Demo.WpfBadFontCrash](https://github.com/walterlv/Walterlv.Demo.WpfBadFontCrash) 

-->