# dotnet C# 通过 Vortice 将 ID2D1CommandList 作为特效的输入源

使用 Direct2D 过程中将可以使用到 Direct2D 强大的特效功能，比如给某些界面绘制内容添加特效支持。本文将告诉大家如何通过 Vortice 将 ID2D1CommandList 作为特效的输入源，从而实现给某些绘制好的界面元素叠加特效

<!--more-->
<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->
<!-- 博客 -->
<!-- 发布 -->

在上一篇 [dotnet C# 通过 Vortice 使用 Direct2D 特效入门](https://blog.lindexi.com/post/dotnet-C-%E9%80%9A%E8%BF%87-Vortice-%E4%BD%BF%E7%94%A8-Direct2D-%E7%89%B9%E6%95%88%E5%85%A5%E9%97%A8.html ) 博客里面告诉大家如何通过 Vortice 使用 Direct2D 特效的入门

在上一篇博客里面是通过将内容绘制在 IWICBitmap 里面，再进行叠加特效的。本文将告诉大家在不使用 IWICBitmap 而是采用 ID2D1CommandList 的方式作为特效的输入源

从 [dotnet C# 通过 Vortice 使用 Direct2D 特效入门](https://blog.lindexi.com/post/dotnet-C-%E9%80%9A%E8%BF%87-Vortice-%E4%BD%BF%E7%94%A8-Direct2D-%E7%89%B9%E6%95%88%E5%85%A5%E9%97%A8.html ) 博客可以知道，特效的输入源是 ID2D1Image 类型。在 Direct2D 里面有许多许多类型都继承于 ID2D1Image 类型，比如上一篇博客用到的 ID2D1Bitmap 类型和本篇的 ID2D1CommandList 类型

换句话说就是只需要将上一篇博客里用到的 IWICBitmap 替换为 ID2D1CommandList 类型即可进行特效的后续对接

在 dotnet 里面通过 Vortice 将 ID2D1CommandList 作为特效的输入源的步骤是

1. 先创建 ID2D1CommandList 对象，在 ID2D1CommandList 进行界面的绘制
2. 创建特效
3. 将 ID2D1CommandList 作为特效的输入源

先来开始第一步，创建 ID2D1CommandList 对象，在 ID2D1CommandList 进行界面的绘制。详细关于如何在 Vortice 使用 ID2D1CommandList 请参阅 [dotnet C# 通过 Vortice 使用 Direct2D 的 ID2D1CommandList 入门](https://blog.lindexi.com/post/dotnet-C-%E9%80%9A%E8%BF%87-Vortice-%E4%BD%BF%E7%94%A8-Direct2D-%E7%9A%84-ID2D1CommandList-%E5%85%A5%E9%97%A8.html )

```csharp
    private static ID2D1CommandList CreateCommandList(ID2D1DeviceContext renderTarget)
    {
        ... // 忽略代码
    }
```

以上的 CreateCommandList 类型将用来创建 ID2D1CommandList 对象，且在 ID2D1CommandList 进行界面的绘制。如以下的例子代码

```csharp
    private static ID2D1CommandList CreateCommandList(ID2D1DeviceContext renderTarget)
    {
        // 随意创建颜色
        var color = new Color4((byte) Random.Shared.Next(255), (byte) Random.Shared.Next(255),
            (byte) Random.Shared.Next(255));

        ID2D1CommandList commandList = renderTarget.CreateCommandList();

        var originTarget = renderTarget.Target;
        renderTarget.Target = commandList;

        using var brush = renderTarget.CreateSolidColorBrush(color);

        for (int i = 0; i < 100; i++)
        {
            renderTarget.FillEllipse(new Ellipse(new System.Numerics.Vector2(Random.Shared.Next(Width), Random.Shared.Next(Height)), 10, 10), brush);
        }

        commandList.Close();

        renderTarget.Target = originTarget;

        return commandList;
    }
```

为了让 CreateCommandList 能够工作起来，需要将其放入到 ID2D1RenderTarget 的 BeginDraw 和 EndDraw 中间调用，如以下代码

```csharp
        // 开始绘制逻辑
        renderTarget.BeginDraw();
        ID2D1CommandList commandList = CreateCommandList(renderTarget);
        renderTarget.EndDraw();
```

获取到 ID2D1CommandList 类型的对象，即可进行第二步，按照 [dotnet C# 通过 Vortice 使用 Direct2D 特效入门](https://blog.lindexi.com/post/dotnet-C-%E9%80%9A%E8%BF%87-Vortice-%E4%BD%BF%E7%94%A8-Direct2D-%E7%89%B9%E6%95%88%E5%85%A5%E9%97%A8.html ) 提供的方法创建特效，如以下代码

```csharp
                var gaussianBlurEffect = d2dDeviceContext.CreateEffect(EffectGuids.GaussianBlur);
                using ID2D1Effect d2dEffect = new ID2D1Effect(gaussianBlurEffect);
```

接下来设置 ID2D1CommandList 类型的对象为 ID2D1Effect 的输入源，如以下代码

```csharp
                ID2D1Image image = commandList;

                d2dEffect.SetInput(0, image, new RawBool(true));
```

如此即可完成将 ID2D1CommandList 作为特效的输入源，接下来只需要设置一些特效的参数，将特效绘制在界面上即可，如以下代码

```csharp
                d2dEffect.SetValue(D2D1_GAUSSIANBLUR_PROP_STANDARD_DEVIATION, count / 60f * 3f);

                renderTarget.DrawImage(d2dEffect, new Vector2(Random.Shared.Next(Width / 100), Random.Shared.Next(Height / 100)));
```


通过这个方法就可以给界面叠加上特效。核心方法就是将界面绘制在 ID2D1CommandList 上，再将 ID2D1CommandList 作为特效输入源，最后将特效绘制在界面上

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/0c5354bdda610102709ce3dc9b6dbeaa6d1b70b8/VorticeD2DCommandListToEffect) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0c5354bdda610102709ce3dc9b6dbeaa6d1b70b8/VorticeD2DCommandListToEffect) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0c5354bdda610102709ce3dc9b6dbeaa6d1b70b8
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0c5354bdda610102709ce3dc9b6dbeaa6d1b70b8
```

获取代码之后，进入 VorticeD2DCommandListToEffect 文件夹

更多 DirectX 和 D2D 以及 Vortice 库的博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

另外，我创建了专门聊 Vortice 的 QQ 群： 622808968 欢迎加入交流技术