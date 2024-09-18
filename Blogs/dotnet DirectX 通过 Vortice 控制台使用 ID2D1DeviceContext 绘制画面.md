---
title: dotnet DirectX 通过 Vortice 控制台使用 ID2D1DeviceContext 绘制画面
description: 在上一篇博客里面告诉大家，如何使用 Vortice 从零开始控制台创建 Direct2D1 窗口。上一篇博客采用的是 CreateDxgiSurfaceRenderTarget 的方式拿到了 ID2D1RenderTarget 进行绘制，本文将和大家介绍另一个方式，通过 ID2D1DeviceContext 绘制画面。从底层来说，这两个方式底层都是相同的，只是上层的 API 调用方法不相同而已

<!--more-->

tags: C# D2D DirectX Vortice Direct2D
category: 
---

<!-- CreateTime:2023/5/22 8:51:31 -->

<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->
<!-- 博客 -->
<!-- 发布 -->

本文属于 DirectX 系列博客，更多 DirectX 和 D2D 以及 Vortice 库的博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

上一篇： [DirectX 使用 Vortice 从零开始控制台创建 Direct2D1 窗口修改颜色](https://blog.lindexi.com/post/DirectX-%E4%BD%BF%E7%94%A8-Vortice-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Direct2D1-%E7%AA%97%E5%8F%A3%E4%BF%AE%E6%94%B9%E9%A2%9C%E8%89%B2.html )

在上一篇博客里面，咱创建了一个 Win32 空窗口，接着给他挂上了 DirectX 交换链。使用以下代码从交换链里面拿到了 DXGI 平面，拿到的的 DXGI 平面即可被绘制 2D 内容在上面，从而将内容绘制输出到窗口上

```csharp
        DXGI.IDXGISwapChain1 swapChain = ... // 忽略交换链之前的代码

        D3D11.ID3D11Texture2D backBufferTexture = swapChain.GetBuffer<D3D11.ID3D11Texture2D>(0);

        // 获取到 dxgi 的平面，这个屏幕就约等于窗口渲染内容
        DXGI.IDXGISurface dxgiSurface = backBufferTexture.QueryInterface<DXGI.IDXGISurface>();
```

在上一篇博客里面，在 IDXGISurface 上绘制 2D 内容的方式是通过 CreateDxgiSurfaceRenderTarget 的方式创建 ID2D1RenderTarget 进而进行通过 ID2D1RenderTarget 绘制界面，如代码

```csharp
        // 对接 D2D 需要创建工厂
        D2D.ID2D1Factory1 d2DFactory = D2D.D2D1.D2D1CreateFactory<D2D.ID2D1Factory1>();

        // 方法1：
        var renderTargetProperties = new D2D.RenderTargetProperties(Vortice.DCommon.PixelFormat.Premultiplied);

        // 在窗口的 dxgi 的平面上创建 D2D 的画布，如此即可让 D2D 绘制到窗口上
        D2D.ID2D1RenderTarget d2D1RenderTarget =
            d2DFactory. (dxgiSurface, renderTargetProperties);

        var renderTarget = d2D1RenderTarget;
```

本文将替换上面代码的方法1部分，按照官方文档提供的另一个方法，通过创建 ID2D1Device 设备，从 ID2D1Device 设备创建出 ID2D1DeviceContext 对象，设置 ID2D1DeviceContext 的输出 Target 为 DXGI 平面，从而使用 ID2D1DeviceContext 绘制界面

<!-- https://learn.microsoft.com/en-us/windows/win32/direct2d/images/devicecontextdiagram.png -->
![](http://cdn.lindexi.site/lindexi%2F2023521105422700.jpg)

按照上图的描述，从创建出来的 Direct3D Device 设备里面，使用 QueryInterface 获取到 DXGI Device 对象。然后分为两个路线，一个是获取 DXGI 的平面，也就是上一篇博客提供的方法，这一路线已完成。另一个就是通过 DXGI Device 设备在 ID2D1Factory1 工厂里创建出 Direct2D Device 设备。再通过 Direct2D Device 设备创建出 Direct2D Device Context 上下文，将 DXGI 的平面转换为 Direct2D Bitmap 作为 Direct2D Device Context 的输出源。如此即可在 Direct2D Device Context 绘制内容输出到窗口界面上

按照上面描述的方法，咱一步步实现

```csharp
        // 获取 DXGI 设备，用来创建 D2D 设备
        DXGI.IDXGIDevice dxgiDevice = d3D11Device.QueryInterface<DXGI.IDXGIDevice>();
```

使用工厂创建 D2D 设备

```csharp
        D2D.ID2D1Factory1 d2DFactory = ...
        // 创建 Direct2D Device 设备
        ID2D1Device d2dDevice = d2DFactory.CreateDevice(dxgiDevice);
```

从设备里面创建 Direct2D Device Context 上下文

```csharp
        // 创建 Direct2D Device Context 上下文
        ID2D1DeviceContext d2dDeviceContext = d2dDevice.CreateDeviceContext();
```

将 DXGI 平面转换为 ID2D1Bitmap1 用于设置到 ID2D1DeviceContext 的输出

```csharp
        ID2D1Bitmap1 d2dBitmap = d2dDeviceContext.CreateBitmapFromDxgiSurface(dxgiSurface);
```

设置 ID2D1Bitmap1 为 ID2D1DeviceContext 的输出

```csharp
        d2dDeviceContext.Target = d2dBitmap;
```

接着将 ID2D1DeviceContext 当成 RenderTarget 即可进行绘制，如以下代码

```csharp
                // 开始绘制逻辑
                renderTarget.BeginDraw();

                // 清空画布
                renderTarget.Clear(new Color4(0xFF, 0xFF, 0xFF));

                // 随意创建颜色
                var color = new Color4((byte) Random.Shared.Next(255), (byte) Random.Shared.Next(255),
                    (byte) Random.Shared.Next(255));
                using var brush = renderTarget.CreateSolidColorBrush(color);
                renderTarget.FillEllipse(new Ellipse(new System.Numerics.Vector2(200, 200), 100, 100), brush);

                renderTarget.EndDraw();
```

以上代码即可在窗口绘制一个有趣颜色的形状

以下是创建 ID2D1DeviceContext 的全部代码

```csharp
       // 获取到 dxgi 的平面，这个屏幕就约等于窗口渲染内容
        DXGI.IDXGISurface dxgiSurface = ...

        // 对接 D2D 需要创建工厂
        D2D.ID2D1Factory1 d2DFactory = D2D.D2D1.D2D1CreateFactory<D2D.ID2D1Factory1>();

        // 方法1：
        //var renderTargetProperties = new D2D.RenderTargetProperties(PixelFormat.Premultiplied);

        //// 在窗口的 dxgi 的平面上创建 D2D 的画布，如此即可让 D2D 绘制到窗口上
        //D2D.ID2D1RenderTarget d2D1RenderTarget =
        //    d2DFactory.CreateDxgiSurfaceRenderTarget(dxgiSurface, renderTargetProperties);
        //var renderTarget = d2D1RenderTarget;

        // 方法2：
        // 创建 D2D 设备，通过设置 ID2D1DeviceContext 的 Target 输出为 dxgiSurface 从而让 ID2D1DeviceContext 渲染内容渲染到窗口上
        // 如 https://learn.microsoft.com/en-us/windows/win32/direct2d/images/devicecontextdiagram.png 图
        // 获取 DXGI 设备，用来创建 D2D 设备
        DXGI.IDXGIDevice dxgiDevice = d3D11Device.QueryInterface<DXGI.IDXGIDevice>();
        ID2D1Device d2dDevice = d2DFactory.CreateDevice(dxgiDevice);
        ID2D1DeviceContext d2dDeviceContext = d2dDevice.CreateDeviceContext();

        ID2D1Bitmap1 d2dBitmap = d2dDeviceContext.CreateBitmapFromDxgiSurface(dxgiSurface);
        d2dDeviceContext.Target = d2dBitmap;

        var renderTarget = d2dDeviceContext;
```

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/afe737c0adf4ed0ef32b4c708d172d63afeaa0ef/VorticeD2DConsoleRender1) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/afe737c0adf4ed0ef32b4c708d172d63afeaa0ef/VorticeD2DConsoleRender1) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin afe737c0adf4ed0ef32b4c708d172d63afeaa0ef
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin afe737c0adf4ed0ef32b4c708d172d63afeaa0ef
```

获取代码之后，进入 VorticeD2DConsoleRender1 文件夹

根据官方文档，继承 ID2D1RenderTarget 的 ID2D1DeviceContext 有更多的功能，比如创建特效等

从这个方面上，其实上一篇博客拿到的 ID2D1RenderTarget 其实和本文所拿到的 ID2D1DeviceContext 是相同的东西，至少从底层上是相同的，也就是在上一篇博客的基础上，通过 QueryInterface 即可从 ID2D1RenderTarget 拿到 ID2D1DeviceContext 对象，如以下代码

```csharp
        var renderTargetProperties = new D2D.RenderTargetProperties(Vortice.DCommon.PixelFormat.Premultiplied);

        // 在窗口的 dxgi 的平面上创建 D2D 的画布，如此即可让 D2D 绘制到窗口上
        D2D.ID2D1RenderTarget d2D1RenderTarget =
            d2DFactory.CreateDxgiSurfaceRenderTarget(dxgiSurface, renderTargetProperties);

        var d2dDeviceContext = d2D1RenderTarget.QueryInterface<ID2D1DeviceContext>();
```

如此可以看到这两篇博客其实只是创建的路线不相同，实际原理是相同的

有伙伴好奇为什么我最近写的是通过 Vortice 调用 DirectX 的博客，而不是通过 SharpDx 或 Silk.NET 调用 DirectX 的博客。这不是我收了 Vortice 的钱或者是和 Vortice 有什么 py 交易哈。其原因是 SharpDx 不维护了，作为 SharpDx 的接任者 Vortice 的行为和 API 都会靠近 SharpDx 许多，我编写起来比较顺手。而 Silk.NET 是对 DirectX 的底层封装，由于是直接底层封装，导致使用 Silk.NET 比较繁琐。尽管使用 Silk.NET 的性能从理论分析上能够比 Vortice 和 SharpDx 更好，但从定量上说，其实好不了多少。我所遇到的几乎所有性能问题，基本都卡在渲染上，而不是调用上，调用上的损耗基本可以忽略。那 Silk.NET 是不是就无用武之地？其实不然，在一些情况下，机器的性能不够业务的需求情况下，能省多少就应该省多少。而且在熟悉整个过程之后，即使将 Vortice 换成 Silk.NET 也只不过是一个体力活而已，将各个 API 进行替换即可。而且有趣的是，可以混合着 Vortice 和 Silk.NET 一起用，只有某些模块才使用 Silk.NET 编写

我创建了专门聊 Vortice 的 QQ 群： 622808968 欢迎加入交流技术
