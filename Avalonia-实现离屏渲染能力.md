
# Avalonia 实现离屏渲染能力

本文将告诉大家如何在 Avalonia 实现跨平台的离屏渲染能力

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

我的需求是拿 Avalonia 当成一些图形画面渲染的框架，准备在 Linux 和 Windows 设备上使用。刚好 Avalonia 做好了图形画面渲染的平台隔离能力，再有提供类 WPF 的布局方式，可以让我制作一些精妙的界面内容

我开始在 GitHub 上搜到 <https://github.com/AvaloniaUI/Avalonia/issues/2174> 这个帖子，一开始我按照 @maxkatz6 介绍的方法，顺利地在 Windows 上使用了 EmbeddableControlRoot 进行离屏渲染

然而以上方法在 Linux 上将会抛出 NotSupportedException 异常，导致完全不可用

我仔细阅读了 <https://github.com/AvaloniaUI/Avalonia/issues/2174> 这个帖子，按照 @kekekeks 提供的方法，尝试自己实现 ITopLevelImpl 接口的方式实现了在 Linux 上也能支持离屏渲染能力

实现的做法如下：如 @kekekeks 所教的方法，咱需要先在 csproj 项目文件里面使用 `<AvaloniaAccessUnstablePrivateApis>true</AvaloniaAccessUnstablePrivateApis>` 用于解决构建问题。随后编写一个名为 OffscreenTopLevelImpl 的类型，继承自 Avalonia.Controls.Embedding.Offscreen.OffscreenTopLevelImplBase 类型，其代码如下

```csharp
class OffscreenTopLevelImpl : OffscreenTopLevelImplBase, ITopLevelImpl
{
    public override IEnumerable<object> Surfaces { get; } = [];
    public override IMouseDevice MouseDevice { get; } = new MouseDevice();
}
```

最后将以上定义的 OffscreenTopLevelImpl 放入到 EmbeddableControlRoot 的构造函数里。请记得为 OffscreenTopLevelImpl 的 ClientSize 赋上合适的尺寸，否则你将得到一张空白渲染的图片内容

后续的部分就和 @maxkatz6 介绍的那样做了，代码如下

```csharp
        var imageFilePath = Path.Join(Path.GetTempPath(), $"{Path.GetRandomFileName()}.png");

        await Dispatcher.UIThread.InvokeAsync(async () =>
        {
            var taskCompletionSource = new TaskCompletionSource();

            // https://github.com/AvaloniaUI/Avalonia/issues/2174#issuecomment-3030306384
            var offscreenTopLevelImpl = new OffscreenTopLevelImpl()
            {
                ClientSize = new Size(1000, 600)
            };
            var embeddableControlRoot = new EmbeddableControlRoot(offscreenTopLevelImpl);
            embeddableControlRoot.Width = 1000;
            embeddableControlRoot.Height = 600;
            var mainView = new MainView();
            mainView.Loaded += (sender, args) =>
            {
                using var renderTargetBitmap = new RenderTargetBitmap(new PixelSize(1000, 600));
                renderTargetBitmap.Render(mainView);
                renderTargetBitmap.Save(imageFilePath);
                taskCompletionSource.SetResult();
            };
            embeddableControlRoot.Content = mainView;

            // 准备离屏渲染工作
            embeddableControlRoot.Prepare(); // 调用此方法会触发 Loaded 事件
            embeddableControlRoot.StartRendering();

            await taskCompletionSource.Task;

            embeddableControlRoot.StopRendering();
            embeddableControlRoot.Dispose();
        });

        return imageFilePath;
```

通过以上方式，就能够在 Linux 和 Windows 上实现离屏渲染的能力。请特别记得给 OffscreenTopLevelImpl 赋值上尺寸，否则将拿到一张空白的图片。本文基于的是 11.3.11 版本的 Avalonia 框架实现，可能后续版本存在差异，如果不知道整体项目如何组织或者无法达成你所需效果，还请按照以下方式拉取本文的代码，尝试运行起来了解效果

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e70a7aaaccbe74fd5301433b1122d52d54dd2024/Workbench/KuhallyediNujewinem) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e70a7aaaccbe74fd5301433b1122d52d54dd2024/Workbench/KuhallyediNujewinem) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e70a7aaaccbe74fd5301433b1122d52d54dd2024
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e70a7aaaccbe74fd5301433b1122d52d54dd2024
```

获取代码之后，进入 Workbench/KuhallyediNujewinem 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。