# Avalonia 笔迹渲染太慢了 用 WPF 做加速层

由于 Avalonia 的渲染延迟非常高，而笔迹应用对渲染实时要求高，我尝试在 Windows 下对 Avalonia 做了很多优化尝试，但都距离 WPF 随便写个笔迹应用慢很多。既然 Avalonia 优化不动，那就用 WPF 做加速层

<!--more-->
<!-- CreateTime:2025/04/19 07:27:29 -->

<!-- 发布 -->
<!-- 博客 -->

由于 Avalonia 的渲染延迟非常高，我尝试优化了几波都改不动，我的伙伴们关于减少渲染延迟的提交也没有被合入到主干，因此我决定采用 WPF 作为加速层用来绘制笔迹

我发现 Avalonia 的合成渲染整个模块的逻辑复杂度很高，啃不动，且越来越认为这个渲染延迟是符合 Avalonia 设计的。即这不是因为代码编写的问题，而是框架设计带来的延迟性。从 Avalonia 官方成员给出的设计图也确实能看到，这是 Avalonia 设计如此。详细请看 <https://github.com/AvaloniaUI/Avalonia/pull/16896#issuecomment-2326397534>

在 Linux 上，使用 X11 直接绘制笔迹的性能也比 Avalonia 绘制的渲染实时性高很多，但如果 Avalonia 肯上 [SHM](https://github.com/AvaloniaUI/Avalonia/pull/17118) 和开启[DirtyRects](https://github.com/AvaloniaUI/Avalonia/pull/16849)优化，还是能接近裸 X11 实时渲染的。在 Windows 上，使用 WPF 随意绘制笔迹的渲染实时性也比 Avalonia 高出很多，但我现在没有找到更多的优化 Avalonia 渲染延迟方法了。我用不准确的测量，能够看到 Avalonia 比 WPF 落后 1-2 帧，有时候最多能落后 5 帧。这里说的落后几帧，不代表 Avalonia 掉帧，而是说对实时响应反馈到界面上的渲染实时性。关于实时性渲染测量，这是另一个大坑，我只是用不准确的测量而已

思路是在 Avalonia 应用上面叠加一个 WPF 窗口，在这个 WPF 窗口里面做笔迹绘制，让 WPF 窗口承载 UWP 笔迹的湿笔迹的概念。抬手时，进行湿笔迹到干笔迹的转换，让笔迹转换为干笔迹在 Avalonia 应用上绘制

核心技术点如下

先采用高性能透明窗口带穿透方式，应用在 WPF 窗口，详细请看 [WPF 制作支持点击穿透的高性能的透明背景异形窗口](https://blog.lindexi.com/post/WPF-%E5%88%B6%E4%BD%9C%E6%94%AF%E6%8C%81%E7%82%B9%E5%87%BB%E7%A9%BF%E9%80%8F%E7%9A%84%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84%E9%80%8F%E6%98%8E%E8%83%8C%E6%99%AF%E5%BC%82%E5%BD%A2%E7%AA%97%E5%8F%A3.html )

此时 WPF 窗口盖在 Avalonia 窗口上，也不会让 WPF 窗口吃掉任何的输入。完全只是让 WPF 窗口作为渲染工具人

按照 [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html) 博客提供的方式，直接用 Stroke 做笔迹绘制核心，只是不同的是在这里咱需要调用 `Stroke.GetGeometry` 方法，获取笔迹路径几何

获取笔迹路径几何是为了从 WPF 的 StreamGeometry 转换为 SKPath 几何路径，让 Avalonia 使用 SKPath 渲染。由于现在我没有找到什么好用的方法进行快速的转换，只好先让 StreamGeometry 序列化为 Path 字符串，再使用 `SKPath.ParseSvgPathData` 转换 Path 字符串。这个过程的性能比较差，好在只有从 WPF 笔迹转换为 Avalonia 笔迹时才使用，只有抬手时才调用，且正常用户写字是不会画长线的，性能比较差也只是比较弱的影响

在 Avalonia 这边封装丢了渲染对齐同步，只好取 CompositionBatch 的 Rendered 完成作为事件，通知 WPF 隐藏笔迹。这个过程中可以看到 Avalonia 的渲染实时性比 WPF 差很多，导致 WPF 笔迹隐藏了，但 Avalonia 笔迹还没渲染出来的闪烁问题

在 Avalonia 里面即使调用完了 Render 方法，再等待 CompositionBatch 的 Rendered 完成，然后通知 WPF 隐藏笔迹。此时 WPF 通过 Dispatcher 调度，设置 InvalidateVisual 等待 UI 线程的 OnRender 调入，调入时不再绘制被隐藏的笔迹，再将数据推送到渲染线程，渲染线程重绘之后在界面显示出来，这个过程里面 WPF 已经将笔迹隐藏了，但 Avalonia 还是没有能将笔迹渲染出来。在这个过程里面，已经是 WPF 落后一帧至少才进行笔迹隐藏的，但就这样 Avalonia 还是追不上

这里也不全是 Avalonia 的问题，我认为底层的 Skia-Angle 部分预计也投毒了，这部分需要有空再来调查。之前粗略使用 GpuViewer 调试不到能够支撑的信息，这也和我没有使用一个简单的应用有关

具体的实现代码很简单，但需要多新建几个项目，本文内容里面只给出关键代码片段，如需要全部的项目文件，可到本文末尾找到本文所有代码的下载方法。本文的代码只是粗略的思路演示，需要再添加很多细节处理才能在正式项目使用

- InkBase 基础的项目，同时被 Avalonia 和 WPF 引用。为什么需要这个项目？因为如果 Avalonia 和 WPF 相互引用，那代码就有很多类型写起来冲突
- FebairwemliwoNajojali 这是 Avalonia 主项目，使用了内部的 [11.2.0-hotfix02](https://github.com/dotnet-campus/Avalonia/releases/tag/11.2.0-hotfix02) 版本，但实际上用官方的版本也行。因为在本文项目里面实际上没有用到 <https://github.com/AvaloniaUI/Avalonia/pull/16896> 带来的提升，完全扔掉 Avalonia 的实时渲染，换成用 WPF 窗口进行笔迹渲染
- WpfInk 使用 WPF 做笔迹渲染的项目
- FebairwemliwoNajojali.Desktop 入口项目，同时引用 Avalonia 和 WPF 项目

在 InkBase 添加一些通用的类型和方法，如 InkId 定义等类型，代码如下

```csharp
public readonly partial record struct InkId(int Value)
{
    public static InkId NewId() => new InkId(_nextId++);

    private static int _nextId = 0;

    public override string ToString()
        => $"InkId={Value}";
}

public readonly record struct InkPoint(InkId Id, double X, double Y, float PressureFactor = 0.5f);

public readonly record struct StandardRgbColor(byte A, byte R, byte G, byte B)
{
    public static StandardRgbColor FromArgb(byte a, byte r, byte g, byte b)
    {
        return new StandardRgbColor(a, r, g, b);
    }

    public static StandardRgbColor Red => new StandardRgbColor(0xFF, 0xFF, 0x00, 0x00);
}

public class SkiaStroke
{
    public SkiaStroke(InkId id, SKPath inkPath)
    {
        Id = id;
        InkPath = inkPath;
    }

    public InkId Id { get; }

    public SKPath InkPath { get; }

    public StandardRgbColor Color { get; init; } = StandardRgbColor.Red;

    public required List<InkPoint> PointList { get; init; }
}
```

定义了抽象接口，这样方便在 Avalonia 项目里面不面对 WPF 项目，而是使用接口

```csharp
public interface IWpfInkLayer
{
    StandardRgbColor Color { set; get; }
    double InkThickness { set; get; }

    void Down(InkPoint screenPoint);
    void Move(InkPoint screenPoint);
    void Up(InkPoint screenPoint);

    event EventHandler<SkiaStroke>? StrokeCollected;

    void HideStroke(SkiaStroke skiaStroke);

    void ToggleShowHideAllStroke();

    SkiaStroke PointListToStroke(InkId id, IReadOnlyList<InkPoint> points);
}

public class WpfForAvaloniaInkingAccelerator
{
    public static WpfForAvaloniaInkingAccelerator Instance { get; } = new WpfForAvaloniaInkingAccelerator();

    public IWpfInkLayer InkLayer { get; set; } = null!;
}
```

在 WPF 项目里面核心就是 WpfInkLayer 类，代码如下

```csharp
public class WpfInkLayer : IWpfInkLayer
{
    public WpfInkLayer(WpfInkWindow inkWindow)
    {
        InkWindow = inkWindow;
    }

    public WpfInkWindow InkWindow { get; }

    private readonly Dictionary<InkId, WpfInkDrawingContext> _dictionary = [];

    public StandardRgbColor Color { set; get; } = StandardRgbColor.Red;
    public double InkThickness { set; get; } = 6;

    public void Render(DrawingContext drawingContext)
    {
        //drawingContext.DrawRectangle(_isBlue ? Brushes.Blue : Brushes.Red, null, new Rect(10, 10, 100, 100));
        _isBlue = !_isBlue;

        foreach (WpfInkDrawingContext context in _dictionary.Values)
        {
            if (context.IsHide)
            {
                continue;
            }

            var stroke = context.Stroke;
            var geometry = stroke.GetGeometry();
            var brush = new SolidColorBrush(context.DrawingAttributes.Color);
            drawingContext.DrawGeometry(brush, null, geometry);
        }
    }

    private bool _isBlue;

    public void Down(InkPoint screenPoint)
    {
        Run(() =>
        {
            var context = new WpfInkDrawingContext(Color,InkThickness);
            _dictionary[screenPoint.Id] = context;
            context.Add(screenPoint);

            InkWindow.InvalidateVisual();
        });
    }

    public void Move(InkPoint screenPoint)
    {
        Run(() =>
        {
            if (_dictionary.TryGetValue(screenPoint.Id, out var context))
            {
                context.Add(screenPoint);

                InkWindow.InvalidateVisual();
            }
        });
    }

    public void Up(InkPoint screenPoint)
    {
        Run(() =>
        {
            if (_dictionary.TryGetValue(screenPoint.Id, out var context))
            {
                context.Add(screenPoint);

                InkWindow.InvalidateVisual();

                var geometry = context.Stroke.GetGeometry();
                var path = geometry.ToString();
                if (path.StartsWith("F1"))
                {
                    path = path.Substring("F1".Length);
                }
                var skPath = SKPath.ParseSvgPathData(path);
                StrokeCollected?.Invoke(this, new SkiaStroke(screenPoint.Id, skPath)
                {
                    Color = context.Color,
                    PointList = context.PointList,
                });
            }
        });
    }

    public event EventHandler<SkiaStroke>? StrokeCollected;
    public void HideStroke(SkiaStroke skiaStroke)
    {
        Run(() =>
        {
            if (_dictionary.TryGetValue(skiaStroke.Id, out var context))
            {
                context.IsHide = !context.IsHide;
            }

            InkWindow.InvalidateVisual();
        });
    }

    public void ToggleShowHideAllStroke()
    {
        Run(() =>
        {
            foreach (var context in _dictionary.Values)
            {
                context.IsHide = !context.IsHide;
            }

            InkWindow.InvalidateVisual();
        });
    }

    public SkiaStroke PointListToStroke(InkId id, IReadOnlyList<InkPoint> points)
    {
        throw new NotImplementedException();
    }

    private void Run(Action action)
    {
        InkWindow.Dispatcher.InvokeAsync(action);
    }
}

class WpfInkDrawingContext
{
    public WpfInkDrawingContext(StandardRgbColor color, double inkThickness)
    {
        var drawingAttributes = new DrawingAttributes()
        {
            Color = color.ToWpfColor(),
            Width = inkThickness,
            Height = inkThickness,
        };
        drawingAttributes.FitToCurve = true;
        DrawingAttributes = drawingAttributes;

        Color = color;
        InkThickness = inkThickness;
    }

    public bool IsHide { get; set; }

    public DrawingAttributes DrawingAttributes { get; }

    public List<InkPoint> PointList { get; } = [];

    public Stroke Stroke
    {
        get
        {
            if (_stroke == null)
            {
                _stroke = new Stroke(StylusPointCollection, DrawingAttributes);
            }

            return _stroke;
        }
    }

    private Stroke? _stroke;

    private StylusPointCollection StylusPointCollection { get; } = new StylusPointCollection();
    public StandardRgbColor Color { get; set; }
    public double InkThickness { get; set; }

    public void Add(InkPoint point)
    {
        PointList.Add(point);

        StylusPointCollection.Add(new StylusPoint(point.X, point.Y, point.PressureFactor));

        _stroke = null;
    }
}

static class StandardRgbColorExtension
{
    public static System.Windows.Media.Color ToWpfColor(this StandardRgbColor color)
    {
        return System.Windows.Media.Color.FromArgb(color.A, color.R, color.G, color.B);
    }
}
```

核心转换如下

```csharp
                var geometry = context.Stroke.GetGeometry();
                var path = geometry.ToString();
                if (path.StartsWith("F1"))
                {
                    path = path.Substring("F1".Length);
                }
                var skPath = SKPath.ParseSvgPathData(path);
                StrokeCollected?.Invoke(this, new SkiaStroke(screenPoint.Id, skPath)
                {
                    Color = context.Color,
                    PointList = context.PointList,
                });
```

这个转换逻辑是有损的，从渲染效果上可以看到 WPF 的 StreamGeometry 渲染出来的效果和 SKPath 渲染出来的效果存在一些偏差。由于两个 UI 框架底层渲染存在一些差异性，可能即使有一个无损转换方法，渲染出来的效果依然也会有差异性

现在 Avalonia 在 Windows 上走的是 Skia->Angle->DX 的渲染路径，而 WPF 走的是 DX 的渲染，也就是最终最底层是相同的，只是 Avalonia 的渲染多了一些中间商，这部分预计是有一些差异的

还没有实现的 PointListToStroke 方法是给橡皮擦准备的，橡皮擦对接这部分逻辑不是本文的重点，于是我就略过了这部分代码，橡皮擦对接是另一个坑

为了承载 WpfInkLayer 控件，需要 WpfInkWindow 窗口，这是一个使用高性能透明的窗口，代码如下

```csharp
public class WpfInkWindow : PerformanceDesktopTransparentWindow
{
    public WpfInkWindow()
    {
        Title = "WpfInk";
        WindowStyle = WindowStyle.None;
        Background = new SolidColorBrush(new Color()
        {
            A = 0x5c,
            R = 0x56,
            G = 0x56,
            B = 0x56
        });
        Background = Brushes.Transparent;
        WindowState = WindowState.Maximized;
        SetTransparentHitThrough();

        _wpfInkLayer = new WpfInkLayer(this);
        WpfForAvaloniaInkingAccelerator.Instance.InkLayer = _wpfInkLayer;
    }

    private readonly WpfInkLayer _wpfInkLayer;

    protected override void OnRender(DrawingContext drawingContext)
    {
        _wpfInkLayer.Render(drawingContext);
    }
}
```

窗口创建时就设置了 WpfForAvaloniaInkingAccelerator 的 InkLayer 属性，于是就可以在 Avalonia 项目里使用

在 FebairwemliwoNajojali.Desktop 启动过程中，新建一个 WPF 的 UI 线程跑起来 WPF 应用，代码如下

```csharp
        var thread = new Thread(() =>
        {
            var application = new WpfApplication
            {
                ShutdownMode = ShutdownMode.OnExplicitShutdown
            };
            application.Startup += (sender, args) =>
            {
                var wpfInkWindow = new WpfInkWindow();
                wpfInkWindow.Show();
            };
            application.Run();
        })
        {
            Name = "WpfInkingAcceleratorThread",
            IsBackground = true
        };
        thread.SetApartmentState(ApartmentState.STA);
        thread.Start();
```

由于 Windows 窗口消息调度的设计，完全让 WPF 和 Avalonia 共用一个 UI 线程也是可以的。但是两个框架繁忙的 UI 线程会相互拖慢，不如就让 WPF 和 Avalonia 独立 UI 线程。独立 UI 线程会导致输入层的调度需要跨线程，又会添加一些损耗和延迟。想要最低延迟，自然是输入层不要放在 Avalonia 这边，将输入层也放在 WPF 上。但这样是降低了延迟，却导致业务逻辑不好编写，需要大家根据业务决定

在 Avalonia 项目的核心是 AvaSkiaInkCanvas 类，代码如下

```csharp
public class AvaSkiaInkCanvas : Control
{
    public AvaSkiaInkCanvas()
    {
        HorizontalAlignment = HorizontalAlignment.Stretch;
        VerticalAlignment = VerticalAlignment.Stretch;
    }

    protected override void OnLoaded(RoutedEventArgs e)
    {
        InkingAcceleratorLayer.StrokeCollected += InkingAcceleratorLayer_StrokeCollected;
    }

    private void InkingAcceleratorLayer_StrokeCollected(object? sender, SkiaStroke e)
    {
        Dispatcher.UIThread.InvokeAsync(() =>
        {
            SkiaStrokeList.Add(e);
            _toClearList.Add(e);
            InvalidateVisual();
        });
    }

    private readonly List<SkiaStroke> _toClearList = [];

    public List<SkiaStroke> SkiaStrokeList { get; } = [];

    private IWpfInkLayer InkingAcceleratorLayer => WpfForAvaloniaInkingAccelerator.Instance.InkLayer;

    private readonly Dictionary<int /*PointerId*/, InkDynamicDrawingContext> _dictionary = [];

    private readonly Dictionary<InkId, InkDynamicDrawingContext> _staticInkDynamicDrawingContextDictionary = [];

    protected override void OnPointerPressed(PointerPressedEventArgs e)
    {
        _dictionary.Add(e.Pointer.Id, new InkDynamicDrawingContext());
        var inkPoint = AddPoint(e);

        InkingAcceleratorLayer.Down(inkPoint);
    }

    protected override void OnPointerMoved(PointerEventArgs e)
    {
        if (_dictionary.TryGetValue(e.Pointer.Id, out var inkDynamicDrawingContext))
        {
            var inkPoint = AddPoint(e);

            InkingAcceleratorLayer.Move(inkPoint);
        }
    }

    protected override void OnPointerReleased(PointerReleasedEventArgs e)
    {
        if (_dictionary.Remove(e.Pointer.Id, out InkDynamicDrawingContext? inkDynamicDrawingContext))
        {
            var inkPoint = AddPoint(e, inkDynamicDrawingContext);

            InkingAcceleratorLayer.Up(inkPoint);

            _staticInkDynamicDrawingContextDictionary[inkDynamicDrawingContext.InkId] = inkDynamicDrawingContext;
        }
    }

    private InkPoint AddPoint(PointerEventArgs e, InkDynamicDrawingContext? inkDynamicDrawingContext = null)
    {
        inkDynamicDrawingContext ??= _dictionary[e.Pointer.Id];
        var currentPoint = e.GetCurrentPoint(this);
        var (x, y) = currentPoint.Position;
        var inkPoint = new InkPoint(inkDynamicDrawingContext.InkId, x, y);
        inkDynamicDrawingContext.PointList.Add(inkPoint);
        return inkPoint;
    }

    public override void Render(DrawingContext context)
    {
        var bounds = Bounds;
        var inkCanvasCustomDrawOperation = new InkCanvasCustomDrawOperation()
        {
            Bounds = bounds,
            SkiaStrokeList = SkiaStrokeList.ToList(),
            ToClearList = _toClearList.ToList(),
        };
        _toClearList.Clear();
        context.Custom(inkCanvasCustomDrawOperation);

        if (ElementComposition.GetElementVisual(this) is { } selfVisual)
        {
            Compositor compositor = selfVisual.Compositor;
            CompositionBatch batch = compositor.RequestCompositionBatchCommitAsync();
            batch.Rendered.ContinueWith(_ =>
            {
                foreach (var skiaStroke in inkCanvasCustomDrawOperation.ToClearList)
                {
                    InkingAcceleratorLayer.HideStroke(skiaStroke);
                }
            });
        }
    }
}
```

以上代码里面的 Render 方法里面，将会等待 CompositionBatch 的 Rendered 进行通知 WPF 隐藏笔迹，但即使 WPF 延迟隐藏笔迹了，也会看到闪烁问题。闪烁问题是因为 Avalonia 还没有完成笔迹绘制但 WPF 已经隐藏了笔迹了，我现在只找到了让 WPF 等待 16.6x3 毫秒之后再隐藏的方式，减少闪烁。但闪烁减少了，又会出现笔迹路径重叠的问题，导致看起来抬手时，笔迹动了一下

具体的 InkCanvasCustomDrawOperation 代码如下

```csharp
class InkDynamicDrawingContext
{
    public InkDynamicDrawingContext()
    {
        InkId = InkId.NewId();
    }
    public InkId InkId { get; }

    public List<InkPoint> PointList { get; } = [];
}

file class InkCanvasCustomDrawOperation : ICustomDrawOperation
{
    public required List<SkiaStroke> SkiaStrokeList { get; init; }
    public required List<SkiaStroke> ToClearList { get; init; }

    public Rect Bounds { get; set; }

    public bool HitTest(Point p)
    {
        return true;
    }

    public void Render(ImmediateDrawingContext context)
    {
        var skiaSharpApiLeaseFeature = context.TryGetFeature<ISkiaSharpApiLeaseFeature>();
        if (skiaSharpApiLeaseFeature == null)
        {
            return;
        }

        using var skiaSharpApiLease = skiaSharpApiLeaseFeature.Lease();
        SKCanvas canvas = skiaSharpApiLease.SkCanvas;
        using var paint = new SKPaint();
        foreach (var skiaStroke in SkiaStrokeList)
        {
            paint.Style = SKPaintStyle.Fill;
            paint.Color = new SKColor(skiaStroke.Color.R, skiaStroke.Color.G, skiaStroke.Color.B, skiaStroke.Color.A);
            paint.IsAntialias = true;

            canvas.DrawPath(skiaStroke.InkPath, paint);
        }
    }

    public bool Equals(ICustomDrawOperation? other)
    {
        return ReferenceEquals(other, this);
    }

    public void Dispose()
    {

    }
}
```

本文的代码是没有考虑触摸点坐标转换的，也没有传递压感参数等，这只是一个用于告诉大家这个思路的粗略代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/4d69ef5536debbc07056aa0c0ef1ec390a03f580/AvaloniaIDemo/FebairwemliwoNajojali) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/4d69ef5536debbc07056aa0c0ef1ec390a03f580/AvaloniaIDemo/FebairwemliwoNajojali) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 4d69ef5536debbc07056aa0c0ef1ec390a03f580
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 4d69ef5536debbc07056aa0c0ef1ec390a03f580
```

获取代码之后，进入 AvaloniaIDemo/FebairwemliwoNajojali 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )