# WPF 使用 SharpDX 在 D3DImage 显示

本文告诉大家如何使用 SharpDX 在 D3DImage 显示。在上一篇[WPF 使用 SharpDX](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX.html )只是使用窗口，也就是无法使用其它的 WPF 控件。所以这一篇就来告诉大家如何使用 WPF 控件和使用 SharpDX 。

<!--more-->
<!-- CreateTime:2018/11/19 15:38:35 -->

<div id="toc"></div>
<!-- 标签：WPF,D2D,DirectX,SharpDX,渲染 -->

本文是一个系列，希望大家从第一篇开始看

 - [WPF 使用 Direct2D1 画图入门](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-Direct2D1-%E7%94%BB%E5%9B%BE%E5%85%A5%E9%97%A8.html )

 - [WPF 使用 Direct2D1 画图 绘制基本图形](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-Direct2D1-%E7%94%BB%E5%9B%BE-%E7%BB%98%E5%88%B6%E5%9F%BA%E6%9C%AC%E5%9B%BE%E5%BD%A2.html )

 - [WPF 使用 SharpDX](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX.html )

 - [WPF 使用 SharpDX 在 D3DImage 显示](https://lindexi.gitee.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX-%E5%9C%A8-D3DImage-%E6%98%BE%E7%A4%BA.html ) 

 - [WPF 使用封装的 SharpDx 控件](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8%E5%B0%81%E8%A3%85%E7%9A%84-SharpDx-%E6%8E%A7%E4%BB%B6.html )

 - [WPF 使用 SharpDx 异步渲染](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E5%BC%82%E6%AD%A5%E6%B8%B2%E6%9F%93.html )

如果只是使用 SharpDX 使用窗口渲染，就无法使用其它的 WPF 控件，实际使用经常只是使用 SharpDX 加快一些渲染，很多元素都是不需要。

如果拿来 HWND 做渲染，那么 WPF 只是提供一个窗口，这和 WPF 的设计，高效而且灵活不符合，所以本文就来告诉大家如何使用 SharpDx 高性能渲染同时使用 WPF 的元素。

微软为了大家方便使用 Direct2D 就添加了 D3DImage ，虽然这个元素不是很好用。

## 介绍

先告诉大家什么是 D3DImage ，这是一个可以和 Direct2D、3D 交互的元素，他是一个 ImageSource ，可以放在 Image 控件显示。

使用 D3DImage 会发送一次内存复制，如果在显卡渲染，那么就会先从显卡获得位图，复制到 D3DImage 作为图片显示到 WPF ，也就是同个位图需要现在显卡渲染，然后复制到内存，让 WPF 渲染图片。

一般使用 D3DImage 都不能拿到比原来好的性能。

那么 D3DImage 有什么用？一般渲染是比较慢的，如果需要使用 Dx12 进行加速，而 WPF 无法使用 dx12 那么就需要使用 dx12 渲染。虽然需要使用内存复制，但是经常使用 dx12 渲染的速度比内存复制然后 WPF 显示的速度快。

在 SharpDX 可以使用 D3DImage 进行离屏渲染，本来 WPF 只能有一个渲染线程，但是使用了 SharpDX 就可以有多个渲染线程，这时通过 dx12 加速，一般渲染速度会比不使用 SharpDX 快。

## 创建控件

首先创建一个 .net framework 4.5 以上的项目。还记得[WPF 使用 Direct2D1 画图入门](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-Direct2D1-%E7%94%BB%E5%9B%BE%E5%85%A5%E9%97%A8.html )说需要使用 x64 才可以编译，实际上 SharpDX 可以使用 AnyCpu ，而且支持 .net framwork 4.5 和以上的项目。所以使用 SharpDx 就比较简单。

打开主页面，创建一个图片

```csharp
        <Grid>
            <Image>
                <Image.Source>
                    <interop:D3DImage x:Name="KsyosqStmckfy"></interop:D3DImage>
                </Image.Source>
            </Image>
        </Grid>
```

从上面可以看到D3DImage的方法，他在 WPF 和其他元素没有不一样的。

因为没有直接从 Direct2D 到 D3D 显示的方法，下面需要告诉大家如何在 D3D11 显示 Direct2D 然后通过相同的格式转 D3D9 最后把缓冲区指针显示。

![](http://image.acmx.xyz/lindexi%2F2018422932386479.jpg)

## D3D 设备

如果需要使用 Direct2D 渲染，需要先创建 D3D11 的设备，因为实际的渲染是通过 3D 渲染。

先引用命名，这样大家直接复制代码就不会不知道使用的是哪个

```csharp
using D2D = SharpDX.Direct2D1;
using SharpDX.Direct3D;
using SharpDX.Mathematics.Interop;
using DXGI = SharpDX.DXGI;
using D3D11 = SharpDX.Direct3D11;
using D3D9 = SharpDX.Direct3D9;
```

在使用之前，需要使用 Nuget 安装，安装方法请看[WPF 使用 SharpDX](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX.html )

创建设备请看下面

```csharp
        var device = new D3D11.Device(DriverType.Hardware, D3D11.DeviceCreationFlags.BgraSupport);
```

因为 D3DImage 需要使用 SetBackBuffer 传入指针，所以通过 D3D11.Texture2D 可以作为指针。

下面来告诉大家如何创建 D3D11.Texture2D ，创建的方法因为需要很多参数，所以代码很多

从 D3D11.Texture2D 的构造函数可以知道，需要传入两个参数 D3D11.Device 和 D3D11.Texture2DDescription ，先创建 D3D11.Texture2DDescription 

```csharp
            var width = Math.Max((int) ActualWidth, 100);
            var height = Math.Max((int) ActualHeight, 100);

            var renderDesc = new D3D11.Texture2DDescription
            {
                BindFlags = D3D11.BindFlags.RenderTarget | D3D11.BindFlags.ShaderResource,
                Format = DXGI.Format.B8G8R8A8_UNorm,
                Width = width,
                Height = height,
                MipLevels = 1,
                SampleDescription = new DXGI.SampleDescription(1, 0),
                Usage = D3D11.ResourceUsage.Default,
                OptionFlags = D3D11.ResourceOptionFlags.Shared,
                CpuAccessFlags = D3D11.CpuAccessFlags.None,
                ArraySize = 1
            };
```

参数大家先直接使用，我这里不告诉大家每个参数是怎么计算

现在创建两个参数就可以创建 D3D11.Texture2D ，创建只需要使用下面代码

```csharp
            var renderTarget = new D3D11.Texture2D(device, renderDesc);

```

## 设置指针

创建好了 D3D11.Texture2D 需要让 D3DImage 显示需要使用 SetBackBuffer 设置。

因为传入 D3D11.Texture2D ，但是 D3DImage 是 dx9 的，所以需要转换一下。

首先转换 Format ，因为 D3D11.Texture2D 使用的是 SharpDX.DXGI.Format 需要转换为 D3D9.Format ，请看下面代码

```csharp
        private static D3D9.Format TranslateFormat(D3D11.Texture2D texture)
        {
            switch (texture.Description.Format)
            {
                case SharpDX.DXGI.Format.R10G10B10A2_UNorm:
                    return D3D9.Format.A2B10G10R10;
                case SharpDX.DXGI.Format.R16G16B16A16_Float:
                    return D3D9.Format.A16B16G16R16F;
                case SharpDX.DXGI.Format.B8G8R8A8_UNorm:
                    return D3D9.Format.A8R8G8B8;
                default:
                    return D3D9.Format.Unknown;
            }
        }

```

除了转换还需要拿到指针

```csharp
        private IntPtr GetSharedHandle(D3D11.Texture2D texture)
        {
            using (var resource = texture.QueryInterface<DXGI.Resource>())
            {
                return resource.SharedHandle;
            }
        }
```

窗口的指针

```csharp
        private static D3D9.PresentParameters GetPresentParameters()
        {
            var presentParams = new D3D9.PresentParameters();

            presentParams.Windowed = true;
            presentParams.SwapEffect = D3D9.SwapEffect.Discard;
            presentParams.DeviceWindowHandle = NativeMethods.GetDesktopWindow();
            presentParams.PresentationInterval = D3D9.PresentInterval.Default;

            return presentParams;
        }
```

实际设置的是 D3D9.Texture ，这个类需要传入 D3D9.Device 和D3D9.PresentParameters，所以才需要上面的代码。

传入 D3D9.Device 需要 D3D9.Direct3DEx ，所以请看代码

```csharp
          var format = TranslateFormat(target);
            var handle = GetSharedHandle(target);

            var presentParams = GetPresentParameters();
            var createFlags = D3D9.CreateFlags.HardwareVertexProcessing | D3D9.CreateFlags.Multithreaded |
                              D3D9.CreateFlags.FpuPreserve;

            var d3DContext = new D3D9.Direct3DEx();
            var d3DDevice = new D3D9.DeviceEx(d3DContext, 0, D3D9.DeviceType.Hardware, IntPtr.Zero, createFlags,
                presentParams);
```

现在可以创建 D3D9.Texture ，通过这个来给指针

```csharp
           var renderTarget = new D3D9.Texture(d3DDevice, target.Description.Width, target.Description.Height, 1,
                D3D9.Usage.RenderTarget, format, D3D9.Pool.Default, ref handle);

            using (var surface = renderTarget.GetSurfaceLevel(0))
            {
                _d3D.Lock();
                _d3D.SetBackBuffer(D3DResourceType.IDirect3DSurface9, surface.NativePointer);
                _d3D.Unlock();
            }
```

这样就设置好了，通过 D3D11.Texture2D 就可以显示出来了。

但是直接使用 D3D11.Texture2D 是无法画出来的，如果需要 D2D.RenderTarget 还需要通过 D3D11.Texture2D 创建 Surface 为缓冲区。

```csharp
            var surface = renderTarget.QueryInterface<DXGI.Surface>();

            var d2DFactory = new D2D.Factory();

            var renderTargetProperties =
                new D2D.RenderTargetProperties(new D2D.PixelFormat(DXGI.Format.Unknown, D2D.AlphaMode.Premultiplied));

            _d2DRenderTarget = new D2D.RenderTarget(d2DFactory, surface, renderTargetProperties);

            device.ImmediateContext.Rasterizer.SetViewport(0, 0,(int)ActualWidth , (int) ActualHeight);
```

## 画出来

下面就来尝试使用 D2D.RenderTarget 画出一个矩形，代码写在 CompositionTarget.Rendering ，画出来的代码和之前的一样

```csharp
     private void CompositionTarget_Rendering(object sender, EventArgs e)
        {
            _d2DRenderTarget.BeginDraw();

            OnRender(_d2DRenderTarget);

            _d2DRenderTarget.EndDraw();

            _d3D.Lock();

            _d3D.AddDirtyRect(new Int32Rect(0, 0, _d3D.PixelWidth, _d3D.PixelHeight));

            _d3D.Unlock();

    
        }

        private void OnRender(D2D.RenderTarget renderTarget)
        {
            var brush = new D2D.SolidColorBrush(_d2DRenderTarget, new RawColor4(1, 0, 0, 1));

            renderTarget.Clear(null);

            renderTarget.DrawRectangle(new RawRectangleF(_x, _y, _x + 10, _y + 10), brush);

            _x = _x + _dx;
            _y = _y + _dy;
            if (_x >= ActualWidth - 10 || _x <= 0)
            {
                _dx = -_dx;
            }

            if (_y >= ActualHeight - 10 || _y <= 0)
            {
                _dy = -_dy;
            }
        }

        private float _x;
        private float _y;
        private float _dx = 1;
        private float _dy = 1;
```

主要和原来不同的是需要 AddDirtyRect 告诉重新渲染，不然不会显示


现在修改一下前台界面，尝试添加一些代码

```csharp
        <Grid>
            <Grid Background="Goldenrod">
                <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center" Text="在图片下方"></TextBlock>
            </Grid>

            <Image>
                <Image.Source>
                    <interop:D3DImage x:Name="KsyosqStmckfy"></interop:D3DImage>
                </Image.Source>
            </Image>

            <Grid Background="Bisque" VerticalAlignment="Bottom">
                <TextBlock Margin="0,10,0,0" HorizontalAlignment="Center"  Text="在图片上方，所以看不见矩形"></TextBlock>
                <TextBlock Margin="0,100,0,100" HorizontalAlignment="Center" VerticalAlignment="Center" Text="欢迎来我博客lindexi.gitee.io"></TextBlock>
            </Grid>
        </Grid>
```

下面就是运行的图片

![](http://img.blog.csdn.net/20180421180424248?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGluZGV4aV9nZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

建议复制一下我的代码，在自己的vs粘贴，尝试跑一下，然后继续看博客。

所有代码

前台界面只有一个控件

```csharp
        xmlns:interop="clr-namespace:System.Windows.Interop;assembly=PresentationCore"

            <Image x:Name="DcwtTmmwvcr">
                <Image.Source>
                    <interop:D3DImage x:Name="KsyosqStmckfy"></interop:D3DImage>
                </Image.Source>
            </Image>
```

先添加 引用

```csharp
using D2D = SharpDX.Direct2D1;
using SharpDX.Direct3D;
using SharpDX.Mathematics.Interop;
using DXGI = SharpDX.DXGI;
using D3D11 = SharpDX.Direct3D11;
using D3D9 = SharpDX.Direct3D9;
```

```csharp
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            _d3D = KsyosqStmckfy;
            Loaded += MainWindow_Loaded;
        }

        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            CreateAndBindTargets();
        }

        private void CompositionTarget_Rendering(object sender, EventArgs e)
        {
            _d2DRenderTarget.BeginDraw();

            OnRender(_d2DRenderTarget);

            _d2DRenderTarget.EndDraw();

            _d3D.Lock();

            _d3D.AddDirtyRect(new Int32Rect(0, 0, _d3D.PixelWidth, _d3D.PixelHeight));

            _d3D.Unlock();

    
        }

        private void OnRender(D2D.RenderTarget renderTarget)
        {
            var brush = new D2D.SolidColorBrush(_d2DRenderTarget, new RawColor4(1, 0, 0, 1));

            renderTarget.Clear(null);

            renderTarget.DrawRectangle(new RawRectangleF(_x, _y, _x + 10, _y + 10), brush);

            _x = _x + _dx;
            _y = _y + _dy;
            if (_x >= ActualWidth - 10 || _x <= 0)
            {
                _dx = -_dx;
            }

            if (_y >= ActualHeight - 10 || _y <= 0)
            {
                _dy = -_dy;
            }
        }

        private float _x;
        private float _y;
        private float _dx = 1;
        private float _dy = 1;

        private D3D9.Texture _renderTarget;

        private D3DImage _d3D;

        private D2D.RenderTarget _d2DRenderTarget;

        private void CreateAndBindTargets()
        {
            var width = Math.Max((int) ActualWidth, 100);
            var height = Math.Max((int) ActualHeight, 100);

            var renderDesc = new D3D11.Texture2DDescription
            {
                BindFlags = D3D11.BindFlags.RenderTarget | D3D11.BindFlags.ShaderResource,
                Format = DXGI.Format.B8G8R8A8_UNorm,
                Width = width,
                Height = height,
                MipLevels = 1,
                SampleDescription = new DXGI.SampleDescription(1, 0),
                Usage = D3D11.ResourceUsage.Default,
                OptionFlags = D3D11.ResourceOptionFlags.Shared,
                CpuAccessFlags = D3D11.CpuAccessFlags.None,
                ArraySize = 1
            };

            var device = new D3D11.Device(DriverType.Hardware, D3D11.DeviceCreationFlags.BgraSupport);

            var renderTarget = new D3D11.Texture2D(device, renderDesc);

            var surface = renderTarget.QueryInterface<DXGI.Surface>();

            var d2DFactory = new D2D.Factory();

            var renderTargetProperties =
                new D2D.RenderTargetProperties(new D2D.PixelFormat(DXGI.Format.Unknown, D2D.AlphaMode.Premultiplied));

            _d2DRenderTarget = new D2D.RenderTarget(d2DFactory, surface, renderTargetProperties);

            SetRenderTarget(renderTarget);

            device.ImmediateContext.Rasterizer.SetViewport(0, 0, (int) ActualWidth, (int) ActualHeight);

            CompositionTarget.Rendering += CompositionTarget_Rendering;
        }

        private void SetRenderTarget(D3D11.Texture2D target)
        {
            var format = TranslateFormat(target);
            var handle = GetSharedHandle(target);

            var presentParams = GetPresentParameters();
            var createFlags = D3D9.CreateFlags.HardwareVertexProcessing | D3D9.CreateFlags.Multithreaded |
                              D3D9.CreateFlags.FpuPreserve;

            var d3DContext = new D3D9.Direct3DEx();
            var d3DDevice = new D3D9.DeviceEx(d3DContext, 0, D3D9.DeviceType.Hardware, IntPtr.Zero, createFlags,
                presentParams);

            _renderTarget = new D3D9.Texture(d3DDevice, target.Description.Width, target.Description.Height, 1,
                D3D9.Usage.RenderTarget, format, D3D9.Pool.Default, ref handle);

            using (var surface = _renderTarget.GetSurfaceLevel(0))
            {
                _d3D.Lock();
                _d3D.SetBackBuffer(D3DResourceType.IDirect3DSurface9, surface.NativePointer);
                _d3D.Unlock();
            }
        }

        private static D3D9.PresentParameters GetPresentParameters()
        {
            var presentParams = new D3D9.PresentParameters();

            presentParams.Windowed = true;
            presentParams.SwapEffect = D3D9.SwapEffect.Discard;
            presentParams.DeviceWindowHandle = NativeMethods.GetDesktopWindow();
            presentParams.PresentationInterval = D3D9.PresentInterval.Default;

            return presentParams;
        }

        private IntPtr GetSharedHandle(D3D11.Texture2D texture)
        {
            using (var resource = texture.QueryInterface<DXGI.Resource>())
            {
                return resource.SharedHandle;
            }
        }

        private static D3D9.Format TranslateFormat(D3D11.Texture2D texture)
        {
            switch (texture.Description.Format)
            {
                case SharpDX.DXGI.Format.R10G10B10A2_UNorm:
                    return D3D9.Format.A2B10G10R10;
                case SharpDX.DXGI.Format.R16G16B16A16_Float:
                    return D3D9.Format.A16B16G16R16F;
                case SharpDX.DXGI.Format.B8G8R8A8_UNorm:
                    return D3D9.Format.A8R8G8B8;
                default:
                    return D3D9.Format.Unknown;
            }
        }
    }

    public static class NativeMethods
    {
        [DllImport("user32.dll", SetLastError = false)]
        public static extern IntPtr GetDesktopWindow();
    }
```

[WPF 使用 SharpDx 画图 1.1-CSDN下载](https://download.csdn.net/download/lindexi_gd/10365799 )

参见：

[在 WinForm 中使用 Direct2D - CYJB - 博客园](http://www.cnblogs.com/cyjb/p/Direct2DControl.html )

[Multithreaded Direct2D Apps (Windows)](https://msdn.microsoft.com/en-us/library/windows/desktop/jj569217(v=vs.85).aspx )

[Improving the performance of Direct2D apps (Windows)](https://msdn.microsoft.com/en-us/library/windows/desktop/dd372260(v=vs.85).aspx )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
