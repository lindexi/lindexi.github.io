# C# 从零开始写 SharpDx 应用 初始化dx修改颜色

本文来告诉大家如何在上一篇博客创建的窗口里面使用 Sharpdx 初始化，然后设置窗口颜色。

<!--more-->
<!-- csdn -->
<!-- 标签：D2D,DirectX,SharpDX -->
<div id="toc"></div>

<!-- 草稿 -->

在[C# 控制台创建 Sharpdx 窗口](https://lindexi.oschina.io/lindexi/post/C-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Sharpdx-%E7%AA%97%E5%8F%A3.html )已经创建了一个窗口，现在需要在这个窗口画一些图片

## 创建资源

第一步是需要添加一个方法 `InitializeDeviceResources` 用来初始化资源

这个方法就是写在`KikuSimairme`类里，关于这个类的代码在[C# 控制台创建 Sharpdx 窗口](https://lindexi.oschina.io/lindexi/post/C-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Sharpdx-%E7%AA%97%E5%8F%A3.html )

```csharp

        // 其他被忽略的代码
        private void InitializeDeviceResources()
        {
        }
```

创建一个可以画出来的类需要先创建显示模式描述，通过显示描述创建交换链描述，交换链描述创建设备和交换链，通过交换链和设备可以创建可以画出来的类，在这个类就可以画出无聊的图形

### 模式描述

首先需要创建一个描述显示模式，请看下面代码

```csharp
using SharpDX.Direct3D;
using SharpDX.DXGI;

// 其他忽略的代码
    class KikuSimairme : IDisposable
// 其他忽略的代码
        private void InitializeDeviceResources()
        {
            ModeDescription backBufferDesc =
                new ModeDescription(Width, Height, new Rational(60, 1), Format.R8G8B8A8_UNorm);

        }
```

通过 ModeDescription 就可以描述，前两个参数是表示缓存的大小，在很多的情况，这个值都和显示的大小相同。

第三个参数就是表示刷新率，这里使用的就是 `60/1` 也就是 60hz 

最后一个参数设置的是像素格式，这里使用 8 位的 RGBA 格式，使用一个无符号的 32 位整数表示

更多关于 ModeDescription 请看 [DXGI_MODE_DESC](https://msdn.microsoft.com/en-us/library/windows/desktop/bb173064(v=vs.85).aspx )

### 交换链描述

下面可以来创建交换链的描述，请使用这个代码

```csharp

// 其他忽略的代码
    class KikuSimairme : IDisposable
// 其他忽略的代码

        private void InitializeDeviceResources()
// 其他忽略的代码
            SwapChainDescription swapChainDesc = new SwapChainDescription()
            {
                ModeDescription = backBufferDesc,
                SampleDescription = new SampleDescription(1, 0),
                Usage = Usage.RenderTargetOutput,
                BufferCount = 1,
                OutputHandle = _renderForm.Handle,
                IsWindowed = true
            };
```

交换链的 ModeDescription 就是上面定义的 backBufferDesc 

SampleDescription 用来表示多重采用，这里设置等级 1 也就是关闭多重采样，参见[DXGI_SAMPLE_DESC structure](https://msdn.microsoft.com/en-us/library/windows/desktop/bb173072(v=vs.85).aspx )

Usage 设置 CPU 访问缓冲的权限，这里设置可以访问 RenderTarget 输出，请看 [DXGI_USAGE](https://msdn.microsoft.com/en-us/library/windows/desktop/bb173078(v=vs.85).aspx )

OutputHandle 获取渲染窗口句柄

IsWindowed 这个值设置是否希望是全屏，如果是 true 就是窗口。现在软件还没写好，所以这时全屏可能就无法退出，我就设置了全屏，本金鱼有两个屏幕，所以可以让软件退出

现在已经创建交换链，但是我里面很多设置没有告诉大家还有哪些可以设置

这里有很多都需要在微软官方才可以看到，因为本文是简单的博客，不会在本文介绍。

### 私有变量

下面可以创建私有变量

```csharp
using D3D11 = SharpDX.Direct3D11;

// 其他忽略的代码

    class KikuSimairme : IDisposable
// 其他忽略的代码

        private D3D11.DeviceContext _d3DDeviceContext;
        private SwapChain _swapChain;
        private D3D11.RenderTargetView _renderTargetView;
```

这里使用了 using 定义了 D3D11 ，这样可以区分一些类，如果有看到我之前的博客，会看到我在很多博客里都使用这个方式

### 创建交换链

准备的代码已经写好，可以创建设备，创建了设备才可以画出

```csharp
using SharpDX.Direct3D;

// 其他忽略的代码
    class KikuSimairme : IDisposable
// 其他忽略的代码

        private void InitializeDeviceResources()
// 其他忽略的代码

            D3D11.Device.CreateWithSwapChain(DriverType.Hardware, D3D11.DeviceCreationFlags.None, swapChainDesc,
                out _d3DDevice, out _swapChain);
            _d3DDeviceContext = _d3DDevice.ImmediateContext;
```

第一个参数 DriverType.Hardware 表示希望使用 GPU 渲染

第二个参数选不使用特殊的方法，参见 [D3D11_CREATE_DEVICE_FLAG enumeration](https://msdn.microsoft.com/en-us/library/windows/desktop/ff476107(v=vs.85).aspx )

第三个参数是输入上面的交换链描述

最后的参数是输出设备和交换链

有了交换链和设备可以在缓冲区画出图形，画图形需要使用`RenderTargetView`，为了在其他函数可以使用，这里需要把这个类写在私有变量

```csharp
        private D3D11.RenderTargetView _renderTargetView;

```

然后在 InitializeDeviceResources 使用下面代码

```csharp
// 其他忽略的代码
    class KikuSimairme : IDisposable
// 其他忽略的代码

        private void InitializeDeviceResources()
// 其他忽略的代码

            using (D3D11.Texture2D backBuffer = _swapChain.GetBackBuffer<D3D11.Texture2D>(0))
            {
                _renderTargetView = new D3D11.RenderTargetView(_d3DDevice, backBuffer);
            }
```

## 修改颜色

如果已经看过了之前的博客，那么知道已经有可以画的类，就可以开始画出。

本文没有告诉大家如何画出线和画出圆形，只是告诉大家初始资源，所以到这里本文就结束了。

但是大家可以看到这时的界面和之前一样，会说我的程序是不是写错了。所以我就简单修改一下界面，创建一个函数 Draw 在这个函数写代码

```csharp
// 其他忽略的代码
    class KikuSimairme : IDisposable
// 其他忽略的代码

        private void Draw()
        {
            _d3DDeviceContext.OutputMerger.SetRenderTargets(_renderTargetView);
            _d3DDeviceContext.ClearRenderTargetView(_renderTargetView, ColorToRaw4(Color.Coral));

            _swapChain.Present(1, PresentFlags.None);

            RawColor4 ColorToRaw4(Color color)
            {
                const float n = 255f;
                return new RawColor4(color.R / n, color.G / n, color.B / n, color.A / n);
            }
        }
```

这里为了画出颜色，使用 ColorToRaw4 的类，因为 RawColor4 是传入颜色是 [0,1]，但是很多代码使用的是[0,255]，为了让颜色比较容易写，我就写了这个类。

在`_d3DDeviceContext.OutputMerger.SetRenderTargets(_renderTargetView);` 设置了刚才创建的`_renderTargetView`激活。

第二句代码` _d3DDeviceContext.ClearRenderTargetView(_renderTargetView, ColorToRaw4(Color.Coral));`清理`_renderTargetView`设置颜色，把他放在第一个缓冲。在 dx 有两个缓冲，一个是看不见的，一个是显示的。第一个缓冲就是显示的，第二个就是在第一个显示的时候画出来，于是不停交换，让用户看到一个画好的缓冲。通过这个方法用户可以看到动画

在 `_swapChain.Present(1, PresentFlags.None);` 是等待垂直同步，在刷新完成在完成这个方法

创建资源和颜色设置代码已经写好，现在需要调用方法

```csharp
// 其他忽略的代码
    class KikuSimairme : IDisposable
    {
        public KikuSimairme()
        {
            _renderForm = new RenderForm();
            _renderForm.ClientSize = new Size(Width, Height);

            InitializeDeviceResources();
        }
// 其他忽略的代码
    }
```

如果有看过之前的博客，会发现有一个方法是空的，现在可以在`RenderCallback`添加代码

```csharp
// 其他忽略的代码
    class KikuSimairme : IDisposable
    {
        private void RenderCallback()
        {
            Draw();
        }
// 其他忽略的代码
    }
```

还需要设置一下清理

```csharp
public void Dispose()
{
    renderTargetView.Dispose();
    swapChain.Dispose();
    d3dDevice.Dispose();
    d3dDeviceContext.Dispose();
    renderForm.Dispose();
}
```

现在按一下 F5 就可以运行，看到一个绿色的窗口

所有代码

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using SharpDX.Direct3D;
using SharpDX.DXGI;
using SharpDX.Mathematics.Interop;
using SharpDX.Windows;
using D3D11 = SharpDX.Direct3D11;

namespace NawbemcemXadre
{
    class KikuSimairme : IDisposable
    {
        /// <inheritdoc />
        public KikuSimairme()
        {
            _renderForm = new RenderForm();
            _renderForm.ClientSize = new Size(Width, Height);

            InitializeDeviceResources();
        }

        private const int Width = 1280;

        private const int Height = 720;

        public void Run()
        {
            RenderLoop.Run(_renderForm, RenderCallback);
        }

        private RenderForm _renderForm;

        private D3D11.Device _d3DDevice;
        private D3D11.DeviceContext _d3DDeviceContext;
        private SwapChain _swapChain;
        private D3D11.RenderTargetView _renderTargetView;

        private void RenderCallback()
        {
            Draw();
        }

        private void InitializeDeviceResources()
        {
            ModeDescription backBufferDesc =
                new ModeDescription(Width, Height, new Rational(60, 1), Format.R8G8B8A8_UNorm);
            
            SwapChainDescription swapChainDesc = new SwapChainDescription()
            {
                ModeDescription = backBufferDesc,
                SampleDescription = new SampleDescription(1, 0),
                Usage = Usage.RenderTargetOutput,
                BufferCount = 1,
                OutputHandle = _renderForm.Handle,
                IsWindowed = true
            };

            D3D11.Device.CreateWithSwapChain(DriverType.Hardware, D3D11.DeviceCreationFlags.None, swapChainDesc,
                out _d3DDevice, out _swapChain);
            _d3DDeviceContext = _d3DDevice.ImmediateContext;

            using (D3D11.Texture2D backBuffer = _swapChain.GetBackBuffer<D3D11.Texture2D>(0))
            {
                _renderTargetView = new D3D11.RenderTargetView(_d3DDevice, backBuffer);
            }
        }

        private void Draw()
        {
            _d3DDeviceContext.OutputMerger.SetRenderTargets(_renderTargetView);
            _d3DDeviceContext.ClearRenderTargetView(_renderTargetView, ColorToRaw4(Color.Coral));

            _swapChain.Present(1, PresentFlags.None);

            RawColor4 ColorToRaw4(Color color)
            {
                const float n = 255f;
                return new RawColor4(color.R / n, color.G / n, color.B / n, color.A / n);
            }
        }

        /// <inheritdoc />
        public void Dispose()
        {
            _renderTargetView.Dispose();
            _swapChain.Dispose();
            _d3DDevice.Dispose();
            _d3DDeviceContext.Dispose();
            _renderForm?.Dispose();
        }
    }
}
```

参见：[SharpDX Beginners Tutorial Part 3: Initializing DirectX - Johan Falk](http://www.johanfalk.eu/blog/sharpdx-beginners-tutorial-part-3-initializing-directx )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
