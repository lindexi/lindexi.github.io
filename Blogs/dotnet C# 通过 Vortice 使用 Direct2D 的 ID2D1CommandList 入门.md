---
title: dotnet C# 通过 Vortice 使用 Direct2D 的 ID2D1CommandList 入门
description: 本文将告诉大家如何通过 Vortice 使用 D2D 的 CommandList 功能

<!--more-->

tags: C# D2D DirectX Vortice Direct2D
category: 
---

<!-- CreateTime:2023/5/30 19:29:44 -->


<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->
<!-- 博客 -->
<!-- 发布 -->

本文属于 DirectX 系列博客，更多 DirectX 和 D2D 以及 Vortice 库的博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

上一篇博客： [dotnet DirectX 通过 Vortice 控制台使用 ID2D1DeviceContext 绘制画面](https://blog.lindexi.com/post/dotnet-DirectX-%E9%80%9A%E8%BF%87-Vortice-%E6%8E%A7%E5%88%B6%E5%8F%B0%E4%BD%BF%E7%94%A8-ID2D1DeviceContext-%E7%BB%98%E5%88%B6%E7%94%BB%E9%9D%A2.html )

在 Direct2D 里面的一个很基础也很重要的功能组件是 D2D 的 CommandList 功能。通过 D2D 的 CommandList 功能，可以用来存放记录一系列的绘制命令，从而将绘制内容作为 ID2D1Image 参与其他功能。比如将继承于 ID2D1Image 的 ID2D1CommandList 作为 ID2D1DeviceContext 的 DrawImage 参数被进行绘制。比如作为特效的输入等等

更多关于 Direct2D 的 ID2D1CommandList 的优势还请自行了解，本文着重在于如何在 Vortice 创建 ID2D1CommandList 对象，以及将绘制内容输入到 ID2D1CommandList 里

在上一篇博客的基础上，创建 CreateCommandList 方法，将在此方法里面编写创建 ID2D1CommandList 对象的逻辑，方法大概如下

```csharp
        ID2D1CommandList CreateCommandList()
        {
            ...
        }
```

创建 ID2D1CommandList 对象需要用到 ID2D1DeviceContext 的 CreateCommandList 方法，代码如下

```csharp
            ID2D1CommandList commandList = renderTarget.CreateCommandList();
```

创建完成之后，需要将 ID2D1DeviceContext 的 Target 挂过去，如以下代码

```csharp
            var originTarget = renderTarget.Target;
            ID2D1CommandList commandList = renderTarget.CreateCommandList();
            renderTarget.Target = commandList;
```

接着即可使用 ID2D1DeviceContext 进行绘制界面，如以下代码在 ID2D1CommandList 里记录绘制界面

```csharp
            using var brush = renderTarget.CreateSolidColorBrush(color);

            // 此时绘制过去的都是在 ID2D1CommandList 里面
            renderTarget.DrawRoundedRectangle(new RoundedRectangle()
            {
                RadiusX = 5,
                RadiusY = 5,
                Rect = new Vortice.RawRectF(100, 100, 600, 300)
            }, brush, 5);

            var backgroundBrush = renderTarget.CreateSolidColorBrush(new Color4(0x64, 0x95, 0xED));

            renderTarget.FillRoundedRectangle(new RoundedRectangle()
            {
                RadiusX = 5,
                RadiusY = 5,
                Rect = new Vortice.RawRectF(115, 115, 590, 290)
            }, backgroundBrush);
```

完成之后别忘了调用 ID2D1CommandList 的 Close 方法，将 ID2D1DeviceContext 的 Target 设置回原先的对象

```csharp
            commandList.Close();

            renderTarget.Target = originTarget;
```

获取到 ID2D1CommandList 之后，可以作为 ID2D1Image 在 ID2D1DeviceContext 使用 DrawImage 进行绘制，代码如下

```csharp
                // 开始绘制逻辑
                renderTarget.BeginDraw();

                // 清空画布
                renderTarget.Clear(new Color4(0xFF, 0xFF, 0xFF));

                using ID2D1CommandList commandList = CreateCommandList();
                ID2D1Image image = commandList;

                renderTarget.DrawImage(image);

                renderTarget.EndDraw();
```

如此即可将 ID2D1CommandList 绘制到画布上

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/08dae017fae148b8eb014a08296ac2f81da218d7/VorticeD2DCommandList) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/08dae017fae148b8eb014a08296ac2f81da218d7/VorticeD2DCommandList) 上，可以通过以下方式获取整个项目的代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 08dae017fae148b8eb014a08296ac2f81da218d7
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 08dae017fae148b8eb014a08296ac2f81da218d7
```

获取代码之后，进入 VorticeD2DCommandList 文件夹
