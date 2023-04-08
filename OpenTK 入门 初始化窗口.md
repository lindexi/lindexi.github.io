# OpenTK 入门 初始化窗口

本文属于 OpenTK 入门博客，这是一项使用 C# 做底层调用 OpenGL 和 OpenAL 和 OpenCL 的技术。但值得一提的是，如果是想做渲染相关的话，当前是不建议使用 OpenGL 的，无论是从性能上还是其他方面，都不具备优势

<!--more-->
<!-- CreateTime:2023/4/7 8:33:59 -->

<!-- 发布 -->
<!-- 博客 -->
<!-- 标签： 渲染 -->

本文将从控制台开始，初始化创建窗口，在窗口里面承载 OpenGL 的内容。创建 OpenGL 空窗口的方式有很多，与之相关的是采用 Silk.NET 库创建 OpenGL 窗口，详细请看 [使用 Silk.NET 创建 OpenGL 空窗口项目例子](https://blog.lindexi.com/post/%E4%BD%BF%E7%94%A8-Silk.NET-%E5%88%9B%E5%BB%BA-OpenGL-%E7%A9%BA%E7%AA%97%E5%8F%A3%E9%A1%B9%E7%9B%AE%E4%BE%8B%E5%AD%90.html )

先新建一个 .NET 7 的控制台应用，接着按照 dotnet 的惯例，安装上 OpenTK 的 NuGet 库。对于 .NET 7 项目，项目文件使用 SDK 风格的 csproj 格式，可以在 VisualStudio 里双击项目进入编辑 csproj 文件，在 csproj 文件里加上以下代码用来安装库

```xml
  <ItemGroup>
    <PackageReference Include="OpenTK" Version="4.7.7" />
  </ItemGroup>
```

修改之后的 csproj 项目文件的代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="OpenTK" Version="4.7.7" />
  </ItemGroup>

</Project>
```

完成安装 NuGet 库之后，进入到 Program.cs 文件里，为了方便后续编写代码，先引用命名空间

```csharp
using OpenTK;
using OpenTK.Graphics.OpenGL4;
using OpenTK.Mathematics;
using OpenTK.Windowing.Common;
using OpenTK.Windowing.Common.Input;
using OpenTK.Windowing.Desktop;
```

接着开始编写一个用来承载 OpenGL 的窗口，代码如下

```csharp
public sealed class MainWindow : GameWindow
{
    public MainWindow(GameWindowSettings gameWindowSettings, NativeWindowSettings nativeWindowSettings) : base(gameWindowSettings, nativeWindowSettings)
    {
    }
}
```

为了体现承载 OpenGL 效果，咱接下来重写 OnRenderFrame 方法，在这个方法里面修改背景色

```csharp
public sealed class MainWindow : GameWindow
{
    public MainWindow(GameWindowSettings gameWindowSettings, NativeWindowSettings nativeWindowSettings) : base(gameWindowSettings, nativeWindowSettings)
    {
    }

    protected override void OnRenderFrame(FrameEventArgs args)
    {
        Title = $"(Vsync: {VSync}) FPS: {1f / args.Time:0}";

        Color4 backColor;
        backColor.A = 1.0f;
        backColor.R = Random.Shared.NextSingle();
        backColor.G = Random.Shared.NextSingle();
        backColor.B = Random.Shared.NextSingle();
        GL.ClearColor(backColor);
        GL.Clear(ClearBufferMask.ColorBufferBit | ClearBufferMask.DepthBufferBit);

        SwapBuffers();
    }
}
```

以上代码可以将 FPS 输出到窗口标题，且修改窗口的背景色

完成窗口定义之后，就可以在 Main 函数跑起来，代码如下

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        var mainWindow = new MainWindow(new GameWindowSettings()
        {
            RenderFrequency = 0,
        }, new NativeWindowSettings()
        {
            Size = new Vector2i(1000, 1000 / 2),
            Title = "OpenTK",
            Vsync = VSyncMode.Off
        });
        mainWindow.Run();
    }
}
```

跑起来的时候设置 RenderFrequency 为 0 就是告诉应用，用尽可能快的刷新速度，再关闭 Vsync 垂直同步即可进行高速刷新

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/6b04d4bc03c379f776ac4d372833ca1118cb9343/NaeawurcaDowhemcenunalfe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6b04d4bc03c379f776ac4d372833ca1118cb9343/NaeawurcaDowhemcenunalfe) 欢迎访问

可以通过如下方式获取源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6b04d4bc03c379f776ac4d372833ca1118cb9343
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6b04d4bc03c379f776ac4d372833ca1118cb9343
```

获取代码之后，进入 NaeawurcaDowhemcenunalfe 文件夹

我创建了一个 OpenTK 技术讨论群： 789025426 欢迎加入

更多渲染相关，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )