# WPF 使用 SharpDX

本文告诉大家如何在 WPF 使用 SharpDX 做绘制，本文是入门级博客

<!--more-->
<!-- CreateTime:2019/3/6 16:52:37 -->

<div id="toc"></div>
<!-- 标签：WPF,D2D,DirectX,SharpDX,渲染 -->

本文属于 [WPF 渲染相关博客](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html ) 系列，系列博客里面按照一定的逻辑顺序排列，方便大家一篇篇阅读
 
先介绍一下 SharpDx ，这是一个底层封装的 DirectX 库，支持 AnyCpu ，支持 Direct3D9, Direct3D11, Direct3D12，Direct2D1。支持 win32 程序和商店程序。但现在（2025） SharpDx 已经停止维护了，可以使用 [SharpDx 的代替项目](https://blog.lindexi.com/post/SharpDx-%E7%9A%84%E4%BB%A3%E6%9B%BF%E9%A1%B9%E7%9B%AE.html ) 列出的替代项目代替

## 环境

需要 .NET 4.5 和以上的环境才可以使用。

## 安装

首先安装 SharpDX 的库，需要安装下面几个库

![](http://cdn.lindexi.site/lindexi%2F2018420916204836.jpg)

## 创建工厂

使用 SharpDX 和 DirectX 一样，开始都需要创建工厂，然后创建RenderTarget，之后才可以显示基础图形。

先引用命名

```csharp
using D2D = SharpDX.Direct2D1;
using WIC = SharpDX.WIC;
using DW = SharpDX.DirectWrite;
using DXGI = SharpDX.DXGI;
```

需要在 Loaded 之后添加代码

```csharp
                var factory = new D2D.Factory();

```

## 创建 RenderTarget 

创建 RenderTarget 可以尝试 WindowRenderTarget ，因为是入门博客，我不告诉大家如何使用其他几个 RenderTarget ，如果想知道，请自己多去看博客。

创建 WindowRenderTarget 需要参数 RenderTargetProperties ，HwndRenderTargetProperties。所以需要先创建这两个。

创建 RenderTargetProperties 需要参数 PixelFormat ，请看下面

```csharp
                var pixelFormat = new D2D.PixelFormat(DXGI.Format.B8G8R8A8_UNorm, D2D.AlphaMode.Straight);

                var renderTargetProperties = new D2D.RenderTargetProperties(D2D.RenderTargetType.Default, pixelFormat,
                    96, 96, D2D.RenderTargetUsage.None, D2D.FeatureLevel.Level_DEFAULT);
```

RenderTargetProperties 需要的参数是 RenderTargetType ，PixelFormat，dpiX，dpiY，RenderTargetUsage，FeatureLevel，参数大家看命名就知道是做什么的，在这里就不告诉大家。

创建 HwndRenderTargetProperties 请看下面代码

```csharp
                var hwndRenderTargetProperties = new D2D.HwndRenderTargetProperties();
                hwndRenderTargetProperties.Hwnd = new WindowInteropHelper(this).Handle;
```

现在尝试创建 RenderTarget 请看代码

```csharp
               var renderTarget = new D2D.WindowRenderTarget(factory, renderTargetProperties, hwndRenderTargetProperties);
```

因为需要拿到 RenderTarget 进行画基础图形，一般把 RenderTarget 放在字段。

```csharp
        public MainWindow()
        {
            InitializeComponent();

            Loaded += (s, e) =>
            {
                var factory = new D2D.Factory();

                var pixelFormat = new D2D.PixelFormat(DXGI.Format.B8G8R8A8_UNorm, D2D.AlphaMode.Straight);

                var hwndRenderTargetProperties = new D2D.HwndRenderTargetProperties();
                hwndRenderTargetProperties.Hwnd = new WindowInteropHelper(this).Handle;
                hwndRenderTargetProperties.Hwnd = new WindowInteropHelper(this).Handle;
                hwndRenderTargetProperties.PixelSize = new Size2((int)ActualWidth, (int)ActualHeight);

                var renderTargetProperties = new D2D.RenderTargetProperties(D2D.RenderTargetType.Default, pixelFormat,
                    96, 96, D2D.RenderTargetUsage.None, D2D.FeatureLevel.Level_DEFAULT);

                _renderTarget = new D2D.WindowRenderTarget(factory, renderTargetProperties, hwndRenderTargetProperties);
            };
        }

        private D2D.RenderTarget _renderTarget;
```

这里的 PixelFormat 使用 B8G8R8A8_UNorm 的意思是每个元素包含4个8位无符号分量，分量的取值范围在[0,1]区间内的浮点数，因为不是任何类型的数据都能存储到纹理中的，纹理只支持特定格式的数据存储。这里的 BGRA 的意思分别是 蓝色（Blue）、绿色（Green）、红色（Red）和 alpha（透明度），其他可以选的格式

 - DXGI_FORMAT_R32G32B32_FLOAT：每个元素包含3个32位浮点分量。 
 - DXGI_FORMAT_R16G16B16A16_UNORM：每个元素包含4个16位分量，分量的取值范围在[0,1]区间内。 
 - DXGI_FORMAT_R32G32_UINT：每个元素包含两个32位无符号整数分量。 
 - DXGI_FORMAT_R8G8B8A8_UNORM：每个元素包含4个8位无符号分量，分量的取值范围在[0,1]区间内的浮点数。 
 - DXGI_FORMAT_R8G8B8A8_SNORM：每个元素包含4个8位有符号分量，分量的取值范围在[−1,1] 区间内的浮点数。 
 - DXGI_FORMAT_R8G8B8A8_SINT：每个元素包含4个8位有符号整数分量，分量的取值范围在[−128, 127] 区间内的整数。 
 - DXGI_FORMAT_R8G8B8A8_UINT：每个元素包含4个8位无符号整数分量，分量的取值范围在[0, 255]区间内的整数

更多概念请看[DirectX11 Direct3D基本概念 - CSDN博客](https://blog.csdn.net/sinat_24229853/article/details/48768829 )

## 画圈

现在就和 [WPF 使用 Direct2D1 画图入门](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-Direct2D1-%E7%94%BB%E5%9B%BE%E5%85%A5%E9%97%A8.html ) 差不多方法来画圈，如何可以画出来，那么就是成功使用 SharpDX 。不要问我为什么用画圈来判断是否可以使用 SharpDX，因为在所有基础的 draw 只有椭圆最耗性能。

还是使用 CompositionTarget 来知道什么时候刷新，在函数添加下面代码

```csharp
            CompositionTarget.Rendering += CompositionTarget_Rendering;

        private void CompositionTarget_Rendering(object sender, EventArgs e)
        {
        }
```

因为画椭圆需要三个参数，第一个是 D2D.Ellipse ，第二个是 Brush ，第三个是线条宽度。

因为 Brush 需要使用刚才的工厂创建，如果不使用工厂创建会异常

先创建 SolidColorBrush 然后创建 D2D.Ellipse

```csharp
            var ellipse = new D2D.Ellipse(new RawVector2(100,100),10,10 );

            var brush = new D2D.SolidColorBrush(_renderTarget, new RawColor4(1, 0, 0, 1));
```

Ellipse 的三个参数是圆心和两个半径，上面是画出半径是 10 的圆。

RawColor4 就是 rgba ，颜色是从 0 到 1 ，对应 WPF 的 RGB 从 0 到 255 ，所以需要转换。

准备好几个参数，可以尝试画出来，在画之前需要使用 BeginDraw 。为什么需要调用这个函数，因为实际上调用 Draw 是不会立刻画出来，而是创建绘制命令，如果渲染是 CPU 渲染，那么就会根据命令让 CPU 在内存渲染。如果渲染 GPU 渲染，就会把命令发到 GPU ，让他渲染。

```csharp
        private void CompositionTarget_Rendering(object sender, EventArgs e)
        {
            var ellipse = new D2D.Ellipse(new RawVector2(100, 100), 10, 10);

            var brush = new D2D.SolidColorBrush(_renderTarget, new RawColor4(1, 0, 0, 1));
            _renderTarget.BeginDraw();

            _renderTarget.DrawEllipse(ellipse, brush, 1);

            _renderTarget.EndDraw();
        }
```

![](http://cdn.lindexi.site/lindexi%2F201842010126671.jpg)

重新告诉大家如何画出。首先拿到窗口，在 WPF 能创建的 WindowRenderTarget 最简单是拿到窗口。因为通过几个属性设置如何渲染，在哪渲染，所以还需要多使用几个属性才可以创建 D2D.WindowRenderTarget 。因为需要一个时机对 WindowRenderTarget 画出，所以我就使用 CompositionTarget 对他进行画出。

上面很多参数都没有详细说明，具体请看这位大佬的[博客](https://blog.csdn.net/X_Jun96?tdsourcetag=s_pctim_aiomsg )

简化一下的全部代码如下

```csharp
using PInvoke;

using SharpDX;
using SharpDX.Direct2D1;

using System.Windows;
using System.Windows.Interop;
using SharpDX.Mathematics.Interop;
using D2D = SharpDX.Direct2D1;
using DXGI = SharpDX.DXGI;
using System.Windows.Media;
using System.Reflection;

namespace LifafaheqearNearkairliraywal;

public class Program
{
    [STAThread]
    public static void Main(string[] args)
    {
        Application application = new Application();
        application.Startup += (s, e) =>
        {
            application.MainWindow.Show();
        };

        Window window = new Window();
        D2DRender render = new D2DRender();
        window.Loaded += (s, e) =>
        {
            render.Init(window);
        };
        
        application.MainWindow=window;
        application.Run();
    }
}

class D2DRender
{
    public void Init(Window window)
    {
        _window = window;

        var factory = new D2D.Factory();

        var pixelFormat = new D2D.PixelFormat(DXGI.Format.B8G8R8A8_UNorm, D2D.AlphaMode.Ignore);

        var renderTargetProperties = new D2D.RenderTargetProperties
        (
              // 默认的行为就是尝试使用硬件优先，否则再使用软件
              D2D.RenderTargetType.Default,
              // 像素格式，对于当前大多数显卡来说，选择 B8G8R8A8 是完全能支持的
              // 而且也方便和其他框架，如 WPF 交互
              pixelFormat,
              dpiX: 96,
              dpiY: 96,
              D2D.RenderTargetUsage.None,
              D2D.FeatureLevel.Level_DEFAULT
        );
        var hwndRenderTargetProperties = new D2D.HwndRenderTargetProperties();
        hwndRenderTargetProperties.Hwnd = new WindowInteropHelper(window).Handle;
        ActualWidth = (int)window.ActualWidth;
        ActualHeight = (int)window.ActualHeight;
        hwndRenderTargetProperties.PixelSize = new Size2(ActualWidth, ActualHeight);

        var renderTarget = new D2D.WindowRenderTarget(factory, renderTargetProperties, hwndRenderTargetProperties);
        _renderTarget = renderTarget;

        window.SizeChanged -= Window_SizeChanged;
        window.SizeChanged += Window_SizeChanged;

        AddRendering();
    }

    private int ActualWidth { set; get; }
    private int ActualHeight { set; get; }

    private void AddRendering()
    {
        CompositionTarget.Rendering -= CompositionTarget_Rendering;
        CompositionTarget.Rendering += CompositionTarget_Rendering;
    }

    public void CompositionTarget_Rendering(object? sender, EventArgs e)
    {
        Render();
    }

    private void Window_SizeChanged(object sender, SizeChangedEventArgs e)
    {
        ArgumentNullException.ThrowIfNull(_window);
        ArgumentNullException.ThrowIfNull(_renderTarget);

        var window = _window;

        ActualWidth = (int) window.ActualWidth;
        ActualHeight = (int) window.ActualHeight;

        _renderTarget.Resize(new Size2(ActualWidth, ActualHeight));
    }

    public void Render()
    {
        var renderTarget = _renderTarget;
        if (renderTarget == null)
        {
            throw new InvalidOperationException();
        }

        renderTarget.BeginDraw();

        renderTarget.Clear(new RawColor4(0.2f,0.5f,0.5f,1));

        var width = Random.Shared.Next(100, 200);
        var height = width;
        var maxWidth = ActualWidth - width;
        var maxHeight = ActualHeight - height;

        var x = Random.Shared.Next(width, maxWidth);
        var y = Random.Shared.Next(height, maxHeight);

        var ellipse = new D2D.Ellipse(new RawVector2(x, y), width, height);

        using var brush = new D2D.SolidColorBrush(_renderTarget, new RawColor4(1, 0, 0, 1));

        renderTarget.FillEllipse(ellipse, brush);

        renderTarget.EndDraw();
    }

    private D2D.WindowRenderTarget? _renderTarget;
    private Window? _window;
}
```

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/a582f328a967af32635cba6d08720341431c9c24/LifafaheqearNearkairliraywal) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/a582f328a967af32635cba6d08720341431c9c24/LifafaheqearNearkairliraywal) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin a582f328a967af32635cba6d08720341431c9c24
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin a582f328a967af32635cba6d08720341431c9c24
```

获取代码之后，进入 LifafaheqearNearkairliraywal 文件夹

更多渲染相关请参阅 [WPF 使用 SharpDx 渲染博客导航](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )