# SharpDx 进入全屏模式

本文告诉大家两个不同的方法进入全屏模式

<!--more-->
<!-- CreateTime:2019/5/31 9:05:36 -->

<!-- csdn -->
<!-- 标签：DirectX,SharpDX,渲染 -->

本文属于 [SharpDx 系列](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html) 博客，建议从头开始读

本文的上一篇是 [C# 从零开始写 SharpDx 应用 初始化dx修改颜色](https://blog.lindexi.com/post/C-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99-SharpDx-%E5%BA%94%E7%94%A8-%E5%88%9D%E5%A7%8B%E5%8C%96dx%E4%BF%AE%E6%94%B9%E9%A2%9C%E8%89%B2.html) 本文的代码将会在上一篇的代码上修改

在 SharpDx 里面可以通过两个方法进入全屏模式，一个是窗口进入全屏，另一个是交换链进入全屏

## 窗口模式

和普通的 WPF 窗口差不多，可以在 RenderForm 创建的时候设置进入全屏模式，全屏就是最大化的窗口同时去掉标题栏，隐藏任务栏

在 Windows 只要整个屏幕的所有像素被填充，那么任务栏将会自动隐藏

在 RenderForm 有一个属性是 IsFullscreen 如果单修改这个属性是没有用的，需要同时设置 AllowUserResizing 属性

```csharp
            _renderForm = new RenderForm();

            _renderForm.WindowState = FormWindowState.Maximized;
            _renderForm.IsFullscreen = true;
            _renderForm.AllowUserResizing = false;
```

这样就可以进入全屏，没有标题栏和任务栏

## 设置透明窗口

在窗口里面可以通过 TransparencyKey 设置某个颜色作为透明颜色，然后通过 AllowTransparency 设置支持透明

```csharp
            _renderForm.TransparencyKey = Color.Transparent;
            _renderForm.AllowTransparency = true;
```

这时画出的透明的颜色就是让窗口透明，透明的部分会命中到后面的元素

和 WPF 的全屏不相同的是，在 SharpDx 里面不会占用屏幕透明的内存，也就是透明部分不需要使用程序进程的内存

## 交换链全屏

如果开启交换链的全屏，那么窗口透明将无法使用，使用交换链透明，在 Windows 的 DWM 将会整个屏幕的绘制交给程序，此时的程序绘制速度能更快

其他的应用有关渲染部分都不会执行，特别是其他的 dx 程序，这样程序可以用到很多的计算。用交换链的全屏可以做到更高的性能，但是在进入的时候都会因为屏幕显示切换出现黑色，现在很少有游戏使用交换链全屏

在上一篇说到的 InitializeDeviceResources 方法里面通过 CreateWithSwapChain 方法创建交换链的下方，可以设置全屏

```csharp
        private void InitializeDeviceResources()
        {
            var backBufferDesc =
                new ModeDescription(Width, Height, new Rational(60, 1), Format.R8G8B8A8_UNorm);

            var swapChainDesc = new SwapChainDescription
            {
                ModeDescription = backBufferDesc,
                SampleDescription = new SampleDescription(1, 0),
                Usage = Usage.RenderTargetOutput,
                BufferCount = 1,
                OutputHandle = _renderForm.Handle,
                IsWindowed = true
            };

            Device.CreateWithSwapChain(DriverType.Hardware, DeviceCreationFlags.None, swapChainDesc,
                out _d3DDevice, out _swapChain);

            // 全屏
            _swapChain.SetFullscreenState(new RawBool(true), null);

            // 忽略代码
        }
```

通过 SetFullscreenState 可以将交换链的渲染信息输出到屏幕，在Windows Vista或更高版本中其实 独占 模式不会让整个程序用到所有的 GPU 资源，因为 GPU 是共享的，但是可以让程序用到更多的资源。清真的程序在其他程序进去全屏的时候，渲染策略会和这个程序最小化一样，不会做实际渲染

[SharpDx 系列](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html)

[SharpDX.DXGI.SwapChain.SetFullscreenState(SharpDX.Mathematics.Interop.RawBool, SharpDX.DXGI.Output) Example](https://www.csharpcodi.com/csharp-examples/SharpDX.DXGI.SwapChain.SetFullscreenState(SharpDX.Mathematics.Interop.RawBool,%20SharpDX.DXGI.Output)/ )

[DirectX Graphics Infrastructure (DXGI) Best Practices - Windows applications](https://docs.microsoft.com/en-us/windows/desktop/direct3darticles/dxgi-best-practices#full-screen_issues?wt.mc_id=MVP )

[DXGI API SharpDX](http://sharpdx.org/wiki/class-library-api/dxgi/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
