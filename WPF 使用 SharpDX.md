# WPF 使用 SharpDX

本文告诉大家如何在 WPF 使用 SharpDX ，只是入门。

<!--more-->
<!-- CreateTime:2019/3/6 16:52:37 -->

<div id="toc"></div>
<!-- 标签：WPF,D2D,DirectX,SharpDX,渲染 -->

本文是一个系列

 - [WPF 使用 Direct2D1 画图入门](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-Direct2D1-%E7%94%BB%E5%9B%BE%E5%85%A5%E9%97%A8.html )

 - [WPF 使用 Direct2D1 画图 绘制基本图形](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-Direct2D1-%E7%94%BB%E5%9B%BE-%E7%BB%98%E5%88%B6%E5%9F%BA%E6%9C%AC%E5%9B%BE%E5%BD%A2.html )

 - [WPF 使用 SharpDX](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX.html )

 - [WPF 使用 SharpDX 在 D3DImage 显示](https://lindexi.gitee.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX-%E5%9C%A8-D3DImage-%E6%98%BE%E7%A4%BA.html ) 

 - [WPF 使用封装的 SharpDx 控件](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8%E5%B0%81%E8%A3%85%E7%9A%84-SharpDx-%E6%8E%A7%E4%BB%B6.html )

 - [WPF 使用 SharpDx 异步渲染](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E5%BC%82%E6%AD%A5%E6%B8%B2%E6%9F%93.html )
 
先介绍一下 SharpDx ，一个底层封装的 DirectX 库，支持 AnyCpu ，支持 Direct3D9, Direct3D11, Direct3D12，Direct2D1。支持 win32 程序和商店程序。

## 环境

需要 .NET 4.5 和以上的环境才可以使用。

## 安装

首先安装 SharpDX 的库，需要安装下面几个库

![](http://image.acmx.xyz/lindexi%2F2018420916204836.jpg)

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

![](http://image.acmx.xyz/lindexi%2F201842010126671.jpg)

重新告诉大家如何画出。首先拿到窗口，在 WPF 能创建的 WindowRenderTarget 最简单是拿到窗口。因为通过几个属性设置如何渲染，在哪渲染，所以还需要多使用几个属性才可以创建 D2D.WindowRenderTarget 。因为需要一个时机对 WindowRenderTarget 画出，所以我就使用 CompositionTarget 对他进行画出。

上面很多参数都没有详细说明，具体请看这位大佬的[博客](https://blog.csdn.net/X_Jun96?tdsourcetag=s_pctim_aiomsg )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
