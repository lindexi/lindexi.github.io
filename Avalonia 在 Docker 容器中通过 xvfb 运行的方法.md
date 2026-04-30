# Avalonia 在 Docker 容器中通过 xvfb 运行的方法

本文将告诉大家如何将 Avalonia 应用部署到 Docker 容器中，使用 Avalonia 作为渲染端，通过离屏渲染输出图像。我将在本文分享在这个过程中遇到的核心坑点以及对应的解决方案

<!--more-->
<!-- CreateTime:2026/04/29 07:01:23 -->

<!-- 发布 -->
<!-- 博客 -->

本文内容由 AI 辅助编写

## 背景

我所在的团队正在使用 Avalonia 作为跨平台的渲染引擎，期望把渲染逻辑放进 Docker 容器里，通过 Web 服务对外提供绘图能力。Avalonia 本身依赖 X11 等桌面环境，而 Docker 容器通常是无桌面的，这就带来了几个挑战。经过一番折腾，终于让 Avalonia 在 Docker 里跑了起来，这里记录一下关键步骤。

核心坑点主要有三个：

1. 需要为容器创建虚拟的 X11 环境（通过 `xvfb`）
2. 容器内缺少中文字体，导致文字变成豆腐块
3. 要使用 `OffscreenTopLevelImplBase` 实现离屏渲染，并且一定得记得设置 `ClientSize` 尺寸

下面我按照实际解决问题的逻辑顺序展开说明。

本文提供的方法非 Headless 无头模式，而是采用 xvfb 创建虚拟的 X11 环境，走真实应用处理逻辑。可以提供可获取的虚拟 X11 环境，和执行桌面客户端的 Avalonia 窗口逻辑。如想了解无头模式的用法，请划到本文末尾

## 一、准备 Docker 基础镜像

为了方便，我会在一份 Dockerfile 里同时完成基础环境、.NET 运行时、X11 和字体的安装。完整文件在文末给出，这里先说核心部分。

首先，我们需要一个基于 Ubuntu 的镜像，安装 `xvfb` 来模拟出 X11 Display 环境。

```dockerfile
FROM ubuntu:24.04

# 安装 xvfb 作为虚拟 X 服务器
RUN apt-get install -y xvfb
ENV DISPLAY=:0
```

后续启动容器时，Avalonia 应用会通过 `DISPLAY` 环境变量连接到这个虚拟显示设备。

安装 .NET 运行时时可以只安装 `aspnetcore-runtime`，因为我们的应用会通过 ASP.NET Core 暴露 HTTP 接口。当然，你也可以换成 `dotnet-runtime`，看实际需要。

```dockerfile
RUN apt-get install -y aspnetcore-runtime-10.0
```

## 二、填补字体的黑洞

Avalonia 渲染文字依赖系统字体，而 Docker 官方镜像为了体积几乎什么都不带。如果直接跑，所有中文字符都会变成方框。我的做法是：

1. 从包管理安装一部分常用字体
2. 从 Windows 机器上把 `C:\Windows\Fonts\` 下的 `ttf`、`ttc` 直接拷贝进镜像（这句是重点！）

```dockerfile
# 安装一些基础字体
RUN apt install -y fonts-ipafont fonts-noto fonts-ubuntu fonts-roboto
RUN fc-cache -fv
```

即便如此，默认的字体覆盖仍然不够。在我的实际项目里，还额外从网上下载了“文泉驿”等中文字体。更好的做法就是上面说的，直接从 Windows 字体目录复制，确保渲染效果与开发机一致。

另外，有些渲染效果依赖于 `libgdiplus`，也一并装好：

```dockerfile
RUN apt-get install -y libgdiplus
```

此外，我还发现安装 `inkscape` 能间接带齐许多图形相关的依赖，省去一个个排查的麻烦：

```dockerfile
RUN apt install -y inkscape
```

## 三、在容器中启动 X11 环境

即使 Dockerfile 里安装了 `xvfb`，程序运行时也需要主动拉起虚拟显示服务。我在 `AvaloniaApp` 项目中写了一个 `LinuxDockerEnvironmentHelper` 来负责这件事。

这个工具类会检查当前是否运行在 Docker 中（通过环境变量 `DOTNET_RUNNING_IN_CONTAINER` 判断），如果是，则用 `Process.Start` 启动 `Xvfb`，并等待它就绪。

```csharp
static class LinuxDockerEnvironmentHelper
{
    public static void EnsureX11Ready()
    {
        if (OperatingSystem.IsLinux() && IsRunningInDocker())
        {
            var display = Environment.GetEnvironmentVariable("DISPLAY");
            if (display is null)
                throw new InvalidOperationException("找不到 DISPLAY 环境变量");

            Process.Start("Xvfb", [display, "-screen", "0", "1920x1080x24"]);

            // 等待 X11 准备好，通过 P/Invoke 检测
            var stopwatch = Stopwatch.StartNew();
            while (stopwatch.Elapsed < TimeSpan.FromSeconds(5))
            {
                var displayHandler = XOpenDisplay(IntPtr.Zero);
                if (displayHandler != 0)
                {
                    XCloseDisplay(displayHandler);
                    return;
                }
                Thread.SpinWait(10);
            }
            throw new NotSupportedException("无法启动 X11 环境");
        }
    }

    // 使用 libX11 判断显示是否可用
    [DllImport("libX11.so.6")]
    private static extern IntPtr XOpenDisplay(IntPtr display);
    [DllImport("libX11.so.6")]
    private static extern int XCloseDisplay(IntPtr display);
}
```

以上的一个小点在于，不能立刻启动 Xvfb 的时候就开始初始 Avalonia 框架，否则将会在底层因为无法打开设备而失败。这就是为什么以上代码需要在循环里面调用 XOpenDisplay 确定环境准备完成的原因

在 Avalonia 入口的 `Main` 方法最开始处调用这个助手，确保后续渲染管线能正常工作。

```csharp
[STAThread]
public static void Main(string[] args)
{
    LinuxDockerEnvironmentHelper.EnsureX11Ready();
    // ...
    BuildAvaloniaApp().StartWithClassicDesktopLifetime(args);
}
```

## 四、使用离屏渲染输出图像

Avalonia 本来是为桌面应用设计的，但在 Docker 里我们不需要真正的窗口，而是想把某个控件渲染成图片。这个需求在我之前的博客 [Avalonia 实现离屏渲染能力](https://blog.lindexi.com/post/Avalonia-%E5%AE%9E%E7%8E%B0%E7%A6%BB%E5%B1%8F%E6%B8%B2%E6%9F%93%E8%83%BD%E5%8A%9B.html ) 里已经提过，当时在 Windows 上能够工作，但 Linux 下需要自己实现 `ITopLevelImpl` 接口。这里我们沿用同样的方式。

首先，在项目文件 `AvaloniaApp.csproj` 里添加以下属性，允许访问 Avalonia 内部还未稳定的 API：

```xml
<AvaloniaAccessUnstablePrivateApis>true</AvaloniaAccessUnstablePrivateApis>
```

然后定义一个 `OffscreenTopLevelImpl` 类，继承自 `OffscreenTopLevelImplBase` 并实现 `ITopLevelImpl`：

```csharp
class OffscreenTopLevelImpl : OffscreenTopLevelImplBase, ITopLevelImpl
{
    public override IEnumerable<object> Surfaces { get; } = [];
    public override IMouseDevice MouseDevice { get; } = new MouseDevice();
}
```

**特别注意：务必给 `ClientSize` 设置正确的尺寸，否则会渲染出一张空白图片！** 这是多次踩坑后总结的血泪教训。

随后在 `AppManager.TakeAsync` 方法里，利用 `EmbeddableControlRoot` 承载我们的视图，并在 `Loaded` 事件里完成截图保存：

```csharp
public async Task<string> TakeAsync()
{
    await App.WaitAppLaunched();

    var imageFilePath = Path.Join(Path.GetTempPath(), $"{Path.GetRandomFileName()}.png");
    await Dispatcher.UIThread.InvokeAsync(async () =>
    {
        var offscreenTopLevelImpl = new OffscreenTopLevelImpl()
        {
            ClientSize = new Size(1000, 600) // 一定要设置尺寸！
        };
        var embeddableControlRoot = new EmbeddableControlRoot(offscreenTopLevelImpl)
        {
            Width = 1000,
            Height = 600
        };
        var mainView = new MainView();
        mainView.Loaded += (sender, args) =>
        {
            using var renderTargetBitmap = new RenderTargetBitmap(new PixelSize(1000, 600));
            renderTargetBitmap.Render(mainView);
            renderTargetBitmap.Save(imageFilePath);
            taskCompletionSource.SetResult();
        };
        embeddableControlRoot.Content = mainView;

        embeddableControlRoot.Prepare(); // 此处会触发 Loaded 事件
        embeddableControlRoot.StartRendering();

        await taskCompletionSource.Task;

        embeddableControlRoot.StopRendering();
        embeddableControlRoot.Dispose();
    });

    return imageFilePath;
}
```

这段代码的关键在于：必须先等待 Avalonia 的 UI 线程就绪（通过 `App.WaitAppLaunched()`），然后在 UI 线程中创建离屏渲染的“窗口”，把业务控件放进去，立刻渲染并保存为 PNG。

## 五、在 ASP.NET Core 中暴露接口

最后，我们用一个极简的 ASP.NET Core Web 应用把渲染能力暴露出去。`Program.cs` 里只加一个 MapGet：

```csharp
app.MapGet("/", async () =>
{
    AppManager appManager = new();
    var imageFilePath = await appManager.TakeAsync();
    return Results.File(imageFilePath, "image/png");
});
```

这样，整个链路就通了：Docker 容器启动 → 拉起 Xvfb 虚拟显示 → Avalonia 应用初始化（包含字体、界面框架）→ Web 请求到来时进行离屏渲染并返回图片。

完整 Dockerfile 和项目代码请按照下文给出的所有代码的下载方法进行下载。你也可以根据自己的需要调整尺寸、字体和渲染逻辑。

## 代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/4fc6e1c6c9511c935bae33186fff35b32101077c/AvaloniaIDemo/JolehenilegeKurhenokoyarfai) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/4fc6e1c6c9511c935bae33186fff35b32101077c/AvaloniaIDemo/JolehenilegeKurhenokoyarfai) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 4fc6e1c6c9511c935bae33186fff35b32101077c
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 4fc6e1c6c9511c935bae33186fff35b32101077c
```

获取代码之后，进入 AvaloniaIDemo/JolehenilegeKurhenokoyarfai 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

## 无头模式

本文提供的是采用虚拟 X11 环境做渲染，这与 Avalonia 内置的无头模式 Headless 有稍微的不同。如果只是做简单的渲染处理，没有涉及到窗口或 X11 环境的情况，那直接采用无头模式即可

直接采用无头模式是无需带 xvfb 虚拟 X11 环境的，但该添加的字体依然还是需要添加。核心逻辑只有在 Program.cs 里面添加 UseSkia 和 UseHeadless 且设置 `UseHeadlessDrawing = false` 采用真实 Skia 做渲染。无需编写 OffscreenTopLevelImpl 相关逻辑，和真实的应用差别仅在于 Program.cs 上。当然，别忘了添加 Avalonia.Headless 库的引用。修改之后的 Program.cs 代码示例如下：

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