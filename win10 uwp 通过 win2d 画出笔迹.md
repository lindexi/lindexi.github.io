# win10 uwp 通过 win2d 画出笔迹

本文告诉大家如何在 UWP 上让 win2d 画出笔迹，通过实际测试发现在 UWP 的笔迹的性能比在 WPF 高很多。但是如果只是使用默认的 InkCanvas 可以做的很少，同时性能也不是特别高，在加上 win2d 才可以做到和来画一样快的性能

<!--more-->
<!-- CreateTime:2018/11/2 20:11:00 -->

<!-- csdn -->
<!-- 标签：uwp,win2d -->

在参加[微软技术暨生态大会 2018](https://walterlv.gitee.io/post/tech-summit-2018.html )听了[邵猛](https://www.cnblogs.com/shaomeng/archive/2018/01/14/8228944.html )大佬的[利用 Windows 新特性开发出更好的手绘视频应用](https://www.cnblogs.com/shaomeng/p/9769270.html )学到了使用 win2d 可以画出笔迹。

在之前我一直在想来画的笔迹性能为什么那么好，现在终于了解到了，于是本文就将具体实现写出来。本文的代码不可以用在实际项目上，因为假设用户都是正常书写

在 UWP 的笔迹有设置对笔迹完全控制，在中文翻译，会将 Ink 翻译为墨迹，本文将 Ink 翻译为笔迹或墨迹。

## 界面

如果想要在 win2d 画出笔迹，还是需要使用 InkCanvas 来收集笔迹，不能直接通过 Pointer 来做。通过测试使用 Pointer 和 InkCanvas 的性能相差在我的设备是 16 ms 左右，需要知道，笔迹的书写过程，相差 16 ms 是一个很大的值。

至于为什么通过 InkCanvas 收集笔迹需要在本文下方告诉大家 InkCanvas 的原理。

因为使用 win2d 需要通过 Nuget 安装，这部分请看[在项目安装win2d](https://lindexi.gitee.io/post/win10-uwp-win2d-%E5%85%A5%E9%97%A8-%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86.html) 本文就直接使用

先引用命名空间 `xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"` 这样就可以在界面通过 canvas 使用高性能的 win2d 来画笔迹

```xml
        <canvas:CanvasControl x:Name="Canvas"/>
        <InkCanvas x:Name="InkCanvas"/>
```

## 笔迹性能原理

为什么通过 InkCanvas 可以拿到很高的性能？

因为在 InkCanvas 使用了不清真的方式实现了从触摸收集点的方法，而使用 Pointer 是通过消息循环给到程序，同时 Pointer 还需要经过路由事件，这样就让笔迹书写速度不够快。如果 InkCanvas 只是更快收到触摸消息，那么也无法做到像现在这么快的速度。尝试写一个空白的 UWP 程序，在里面添加笔迹控件，在移动的过程中，进入断点，这时你还可以继续在 UWP 应用上画。

也就是 InkCanvas 的书写和 UWP 的主线程是分开的

在 UWP 的笔迹渲染是分为三个过程，第一个过程是跟随，也就是将当前的点和上一个点直接连出一条线。第二个过程是动态笔迹层，在书写过程就是进行动态笔迹渲染，这时将使用最快的方式画出笔迹。第三个过程是静态笔迹，在 UWP 官方是 Drying 将动态笔迹成为湿笔迹，就像使用钢笔写的一样。而从湿到干就是动态转静态的笔迹。将笔迹转为静态就可以让笔迹变为一个界面元素，参与界面的变化，如选择和层级这些业务。

在动态笔迹只是做渲染，用最快的算法从触摸收集到的点画出来，而静态笔迹就是将动态笔迹转换为普通的元素，可以用来做业务

当然大家也不会关注为什么笔迹在 UWP 那么快，于是就继续在后台代码添加设置。我才不告诉大家，我也不知道他是怎么做的

## 完全控制墨迹

在 UWP 的笔迹可以通过调用 ActivateCustomDrying 方法完全控制笔迹的静态渲染，也就是 InkCanvas 可以让代码处理从动态转静态的方法

```csharp
                _inkSynchronizer = InkCanvas.InkPresenter.ActivateCustomDrying();
```

这里需要使用字段 `_inkSynchronizer` 记录笔迹

## 多指输入

原来的 InkCanvas 不支持多指输入，通过下面的代码可以让 InkCanvas 支持多笔

```csharp
        InkCanvas.InkPresenter.SetPredefinedConfiguration(InkPresenterPredefinedConfiguration
                    .SimpleMultiplePointer);
```

## 转换笔迹

转换笔迹的时候需要在 win2d 上画出静态笔迹

```csharp
Canvas.Draw += CanvasControl_Draw
```

在这个函数里面可以通过 win2d 画出任意的内容

但是需要知道在什么时候开始画，同时 win2d 需要调用 Invalidate 刷新，在笔迹的一笔画完之后可以通过 `InkPresenter_StrokesCollected` 事件拿到添加的笔迹

```csharp
            InkCanvas.InkPresenter.StrokesCollected += InkPresenter_StrokesCollected;
       
        private void InkPresenter_StrokesCollected(InkPresenter sender, InkStrokesCollectedEventArgs args)
        {

        }
```

在 `InkPresenter_StrokesCollected` 就可以将笔迹从动态转换为静态在 Win2d 添加用户的元素

通过 `_inkSynchronizer.BeginDry()` 拿到现在的所有动态的笔迹

从函数上可以看到有 BeginDry 应该就有 EndDry 尝试两个函数一起调用，会发现调用了 EndDry 之后动态笔迹就消失了。如果这时还没有将静态笔迹画出来，界面就看不到原来的笔迹

在 UWP 可以多次调用 BeginDry 拿到动态笔迹，假如现在有动态笔迹 1、2 调用 BeginDry 会返回动态笔迹 1、2 然后用户继续触摸，在界面有动态笔迹 3 再次调用BeginDry会返回第三条笔迹。

但是只能调用一次 EndDry 也就是在调用多次 BeginDry 只有只能调用一次 EndDry 不能相邻两次调用 EndDry 方法

在 win2d 画静态笔迹首先需要刷新界面

```csharp
 private void InkPresenter_StrokesCollected(InkPresenter sender, InkStrokesCollectedEventArgs args)
 {
            _pendingDry = inkSynchronizer.BeginDry();

            Canvas.Invalidate();
 }
```

在 Canvas 的刷新函数画出笔迹

```csharp
 using (CanvasDrawingSession ds = sender.CreateDrawingSession())
 {
     ds.DrawInk(_pendingDry);
 }
```

## 无限漫游

如果现有做无限漫游，可以使用 [CanvasVirtualControl](https://lindexi.gitee.io/post/win10-uwp-win2d-CanvasVirtualControl.html ) 做一个超级大的画布，同时只画出可见的范围

这样可以做到无限漫游添加很多笔迹而软件不会变卡

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
