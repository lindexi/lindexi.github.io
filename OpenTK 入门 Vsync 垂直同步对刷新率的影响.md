# OpenTK 入门 Vsync 垂直同步对刷新率的影响

本文将和大家介绍 Vsync 垂直同步的开启对 OpenTK 应用的刷新率的影响

<!--more-->
<!-- 发布 -->
<!-- 博客 -->
<!-- 标签： 渲染 -->

在上一篇博客 [OpenTK 入门 初始化窗口](https://blog.lindexi.com/post/OpenTK-%E5%85%A5%E9%97%A8-%E5%88%9D%E5%A7%8B%E5%8C%96%E7%AA%97%E5%8F%A3.html ) 告诉了大家如何初始化 OpenTK 承载 OpenGL 的窗口的应用，在上一篇博客基础上，咱尝试修改创建 MainWindow 的参数，从而测试 Vsync 垂直同步对刷新率的影响

回顾上一篇博客提到的代码，创建窗口的时候设置了两个重要参数，分别是 RenderFrequency 和 Vsync 参数

```csharp
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
```

在上一篇博客将 FPS 打在了窗口标题上，运行代码可以看到窗口在不断刷新。在我的电脑上以上代码运行的 FPS 能到 1000 以上

在 OpenTK 里面的 RenderFrequency 表示渲染的频率，设置为 0 表示让 OpenGL 以尽可能快的速度进行刷新。设置 Vsync 为关闭即可关闭垂直同步

接下来尝试修改一下参数，使用以下代码关闭垂直同步

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
            Vsync = VSyncMode.On
        });
        mainWindow.Run();
    }
}
```

开启垂直同步之后，可以看到窗口标题上的 FPS 只能到 60 左右。但是开启之后在我的电脑上的整体效果会更好一些

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