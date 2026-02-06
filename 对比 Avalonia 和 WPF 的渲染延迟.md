# 对比 Avalonia 和 WPF 的渲染延迟

最近我在摸索 Avalonia 的渲染层，这个问题源自于 7 年前，我尝试给 Avalonia 添加笔迹应用。在去年的时候，我发现 Avalonia 的笔迹性能非常糟糕，今年我设计了一个测试用例。在 Avalonia 窗口上叠加一个透明的 WPF 窗口，从 Avalonia 收到鼠标或触摸输入之后，再发送到 WPF 窗口上，让 Avalonia 和 WPF 窗口同时对一个 Border 进行 RenderTransform 平移

<!--more-->
<!-- CreateTime:2026/02/06 07:18:27 -->

<!-- 发布 -->
<!-- 博客 -->

此测试发现了 WPF 的渲染非常跟输入，而 Avalonia 明显落后

在我的测试用例里面，特别让 Avalonia 窗口去接收输入，让 Avalonia 驱动 WPF 的界面。如此可以排除 Avalonia 的输入层带来的延迟。完全只对比 Avalonia 和 WPF 的渲染层

详细请参阅： <https://github.com/AvaloniaUI/Avalonia/discussions/20562>

实验情况如下图所示，蓝色为 Avalonia 的控件，红色是 WPF 的控件

<!-- ![](image/Avalonia 简易对比不同的 Win32CompositionMode 的性能情况/Avalonia 简易对比不同的 Win32CompositionMode 的性能情况0.gif) -->
![](http://cdn.lindexi.site/lindexi-RenderingLatency.gif)

本次测试用的 Avalonia 版本为 11.3.11 版本

具体实验设计如下：

- 新建 WPF 和 Avalonia 空白项目
- 分别在 WPF 和 Avalonia 项目的界面添加不同颜色的 Border 控件。且附加上 TranslateTransform 到 `Border.RenderTransform` 属性上，用于让 Border 控件被移动
- 让 WPF 窗口背景透明，且通过 `GWL_HWNDPARENT` (SetOwner) 让 WPF 窗口显示在 Avalonia 窗口之上。如此运行项目可在同一屏幕上看到 Avalonia 和 WPF 框架的两个窗口，其中 WPF 窗口作为透明窗口叠加在 Avalonia 窗口之上
- 在 Avalonia 框架内监听 Pointer 的按下和移动，当移动的时候设置 Border 的 TranslateTransform 的 X 坐标。随后将此坐标变更发送给到 WPF 应用，让 WPF 应用内的 Border 也做相同的 TranslateTransform 变换
- 脱离 Visual Studio 独立运行项目，使用鼠标或触摸移动 Border 控件

本文所采用的测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/3f38106c92a2a645e0df7b27731b5f84f3e16f78/AvaloniaIDemo/HohaychukeajeherLelfeajune) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/3f38106c92a2a645e0df7b27731b5f84f3e16f78/AvaloniaIDemo/HohaychukeajeherLelfeajune) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 3f38106c92a2a645e0df7b27731b5f84f3e16f78
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 3f38106c92a2a645e0df7b27731b5f84f3e16f78
```

获取代码之后，进入 AvaloniaIDemo/HohaychukeajeherLelfeajune 文件夹，即可获取到源代码

回答一些疑惑：

- 按照以上的实验设计，输入层是从 Avalonia 框架来的，意味着 Avalonia 在输入处理方面还比 WPF 执行得更早。至少 WPF 需要等待 Dispatcher 调度之后才能收到输入
- 无论是 WPF 还是 Avalonia 框架，都执行了相同的上层逻辑代码，都通过 RenderTransform 进行变换
- 为什么不使用动画而是使用输入来测试？因为 WPF 和 Avalonia 的动画模块有很大的实现差异，为了控制变量，选择使用输入来做
- 选择让 WPF 窗口是透明的，如此可以让 WPF 框架承担透明窗口带来的更多渲染代价。在 WPF 框架承受了透明窗口渲染代价的前提下，依然能够轻松领先于 Avalonia 框架，如此更可说明 Avalonia 框架的渲染延迟性
- 在实际实验中，在 4K 分辨率的触摸屏上，能够感受到更大的差异，可见 Avalonia 更落后于 WPF 框架
- 是否设置 Avalonia 使用 LowLatencyDxgiSwapChain 能够解决此问题？设置 LowLatencyDxgiSwapChain 只能缓解问题，但依然在渲染延迟上落后与 WPF 框架