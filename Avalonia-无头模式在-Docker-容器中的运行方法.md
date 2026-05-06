
# Avalonia 无头模式在 Docker 容器中的运行方法

本文将告诉大家如何使用 Avalonia 官方提供的 Headless 库，在 Docker 容器中无需配置 xvfb 运行环境、无需手动实现复杂的离屏渲染适配逻辑，即可快速实现 Avalonia 应用的渲染输出能力。和我之前写的 Avalonia 在 Docker 容器中通过 xvfb 运行的方法提到的实现方式相比，这种方案依赖更少、部署更简单，非常适合服务端纯渲染的业务场景。

<!--more-->


<!-- CreateTime:2026/04/30 08:44:46 -->

<!-- 发布 -->
<!-- 博客 -->

本文内容由 AI 辅助编写

我之前如果在 Linux 或者 Docker 容器中运行 Avalonia 完成渲染任务，得搭建 xvfb 虚拟桌面环境安装大量桌面依赖，以及得自己实现 ITopLevelImpl 接口做底层离屏渲染适配。我在兔基兔基的 Avalonia 交流群（群号： 397510870）里面发了 [Avalonia 在 Docker 容器中通过 xvfb 运行的方法](https://blog.lindexi.com/post/Avalonia-%E5%9C%A8-Docker-%E5%AE%B9%E5%99%A8%E4%B8%AD%E9%80%9A%E8%BF%87-xvfb-%E8%BF%90%E8%A1%8C%E7%9A%84%E6%96%B9%E6%B3%95.html ) 这篇博客，兔基兔基群主教会了我用 Headless 无头模式进行渲染

<!-- [Avalonia 在 Docker 容器中通过 xvfb 运行的方法 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19941247 ) -->

本文介绍的无头模式与 xvfb 配合离屏渲染最大的不同点在于：采用虚拟 X11 环境时，可以跑真实真实应用处理逻辑。可以提供可获取的虚拟 X11 环境，和执行桌面客户端的 Avalonia 窗口逻辑。采用无头模式时，不要求运行环境中存在 X11 环境，也不会受到窗口化的环境干扰。各有优劣点，具体采用哪个方案，取决于业务

在我实际的产品需求里面，我是拿原本的一个桌面端应用程序，通过 ASP.NET Core 包装成服务的方式。原本的桌面端应用存在较多的 X11 调用以及窗口相关逻辑，导致了直接采用无头模式时，行为不符合预期。于是我就采用了配置虚拟 X11 环境的方式

接下来将和大家介绍如何采用无头模式

首先第一步是在项目中安装 Avalonia.Headless 的 NuGet 包，只需要在 csproj 文件中添加如下引用即可，版本可以根据你项目使用的 Avalonia 版本进行匹配：

```xml
<PackageReference Include="Avalonia.Headless" Version="11.3.12" />
```

接下来只需要修改 Program.cs 中的 Avalonia 配置逻辑，这里有两个核心配置是必须的，也是很多开发者容易踩的坑：一是要显式调用 UseSkia 方法启用 Skia 渲染后端，二是在调用 UseHeadless 方法时将 UseHeadlessDrawing 属性设置为 false。如果不配置这两项，Avalonia 会采用 Stub 模式模拟渲染流程，不会执行真实的渲染逻辑，最终得到的图片会是全空白的。

完整的 Program.cs 配置代码如下：

```csharp
class Program
{
    // Initialization code. Don't use any Avalonia, third-party APIs or any
    // SynchronizationContext-reliant code before AppMain is called: things aren't initialized
    // yet and stuff might break.
    [STAThread]
    public static void Main(string[] args) => BuildAvaloniaApp()
        .StartWithClassicDesktopLifetime(args);

    // Avalonia configuration, don't remove; also used by visual designer.
    public static AppBuilder BuildAvaloniaApp()
        => AppBuilder.Configure<App>()
            .UseSkia()
            .UseHeadless(new AvaloniaHeadlessPlatformOptions()
            {
                UseHeadlessDrawing = false,
            })
            .WithInterFont()
            .LogToTrace();
}
```

以上代码中其他的配置和常规 Avalonia 桌面项目没有区别，UseHeadless 方法会自动帮我们完成无头环境下的平台适配，不需要我们再手动处理窗口、渲染设备之类的底层逻辑。

配置完成之后，渲染输出图片的逻辑就和常规的 Avalonia 离屏渲染完全一致了，比如我们可以在主窗口的 Loaded 事件中添加如下逻辑，测试渲染输出的效果：

```csharp
    private void MainWindow_Loaded(object? sender, RoutedEventArgs e)
    {
        if (Design.IsDesignMode)
        {
            return;
        }

        using var renderTargetBitmap = new RenderTargetBitmap(new PixelSize(1920, 1080));
        renderTargetBitmap.Render(this);

        var imageFile = Path.GetFullPath($"{Path.GetRandomFileName()}.png");
        renderTargetBitmap.Save(imageFile);
        Console.WriteLine($"ImageFile={imageFile}， WH={Bounds.Width:0.00}x{Bounds.Height:0.00}");
    }
```

以上逻辑会在窗口加载完成之后，将当前窗口内容渲染为 1920*1080 分辨率的 PNG 图片，保存到临时路径下并输出到控制台。这里加了设计模式的判断，避免在 Visual Studio 设计器中运行时触发逻辑导致异常。你也可以将 Render 方法的参数替换为任意自定义的控件，不需要挂载到窗口也可以完成渲染。

最后在 Docker 中部署的时候，也不需要额外配置 xvfb 相关的依赖，只需要使用包含 Skia 运行时的 .NET 基础镜像即可，相比传统方式镜像体积更小，部署步骤也更简单。如果你的应用不需要其他原生依赖，甚至可以直接使用官方的 .NET 运行时镜像，不需要额外安装任何其他软件包。

通过以上的配置，你就可以快速在 Docker 容器中部署 Avalonia 应用，完成自动化截图、报表生成、动态海报渲染之类的任务，不用再处理复杂的环境适配和底层渲染逻辑。

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e3af9a9a4f7ddd724316450913e6cbe202ef4775/AvaloniaIDemo/WulaycehaRerwurlarrurburkerejea) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e3af9a9a4f7ddd724316450913e6cbe202ef4775/AvaloniaIDemo/WulaycehaRerwurlarrurburkerejea) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e3af9a9a4f7ddd724316450913e6cbe202ef4775
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e3af9a9a4f7ddd724316450913e6cbe202ef4775
```

获取代码之后，进入 AvaloniaIDemo/WulaycehaRerwurlarrurburkerejea 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。