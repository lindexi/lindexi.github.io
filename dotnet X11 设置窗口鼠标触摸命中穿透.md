# dotnet X11 设置窗口鼠标触摸命中穿透

本文记录如何在 X11 应用里面，使用 XShapeCombineRegion 方法配置一个 X11 窗口支持和 Win32 窗口一样的命中测试穿透功能，即对应 Win32 的 WS_EX_TRANSPARENT 的鼠标、触摸等的点击等动作的穿透功能，可以实现在窗口中挖空一块范围直接穿透到后面的窗口

<!--more-->
<!-- CreateTime:2024/05/21 17:01:25 -->

<!-- 发布 -->
<!-- 博客 -->

在 X11 窗口中，想要实现让 X11Window 窗口不可命中，即所有的鼠标、触摸等的事件点击穿透到后面的窗口上，可以采用 libXext.so 提供的 XShapeCombineRegion 方法，也可以使用有[争议](https://en.wikipedia.org/wiki/XFixes)的 libXfixes.so 提供的 XFixesSetWindowShapeRegion 方法

通过以上两个方法即可让 X11 窗口不响应鼠标或触摸的点击输入，让其输入到窗口后面的窗口。适合用来制作一个仅用来展示渲染的窗口，让这个窗口不参与到交互里面

使用比较有争议的 libXfixes.so 提供的 XFixesSetWindowShapeRegion 方法的示例代码如下

```csharp
// 以下的 childWindowHandle 是一个 X11 窗口

var region = XFixesCreateRegion(display, 0, 0);
XFixesSetWindowShapeRegion(display, childWindowHandle, ShapeInput, 0, 0, region);

    [DllImport("libXfixes.so.3")]
    public static extern IntPtr XFixesCreateRegion(IntPtr display, IntPtr rectangles, int nrectangles);

    [DllImport("libXfixes.so.3")]
    public static extern void XFixesSetWindowShapeRegion(IntPtr display, IntPtr window, int shape_type, int x_offset,
        int y_offset, IntPtr region);
```

采用 libXext.so 提供的 XShapeCombineRegion 方法的示例代码如下

```csharp
var region = XCreateRegion();
XShapeCombineRegion(display, childWindowHandle, ShapeInput, 0, 0, region, ShapeSet);

        [DllImport(libX11)]
        public static extern IntPtr XCreateRegion();

        [DllImport("libXext.so.6")]
        public static extern void XShapeCombineRegion(IntPtr display, IntPtr dest, int destKind, int xOff, int yOff, IntPtr region, int op);
```

我尝试创建两个窗口，其中一个窗口调用了 XShapeCombineRegion 方法，运行程序，将设置了的 XShapeCombineRegion 的窗口激活作为前台窗口，点击此窗口的内容，可以看到点击穿透到后面的窗口。如此即可使用 C# dotnet 在 X11 应用里实现 Click Through 窗口点击穿透功能

以上两个方法都能实现功能，且通过阅读 [X Server](https://github.com/XQuartz/xorg-server)的代码，可以发现以上两个方法核心实现基本相同。为了可能的坑点在于 libXfixes.so 可能在某些系统上被砍掉。只是这个 libXfixes.so 也足够旧了，基本上系统都会带的

所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/67cd9188399e7f45bfe83e1af9daf10236b3171c/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/67cd9188399e7f45bfe83e1af9daf10236b3171c/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 67cd9188399e7f45bfe83e1af9daf10236b3171c
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 67cd9188399e7f45bfe83e1af9daf10236b3171c
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

以上代码经过我在 UOS 系统上测试通过，在 UOS 上的 KWin_X11 能够符合预期工作

如运行代码提示找不到 libXext.so 文件，错误内容如下

```
System.DllNotFoundException: Unable to load shared library 'libXext.so' or one of its dependencies. In order to help diagnose loading problems, consider using a tool like strace. If you're using glibc, consider setting the LD_DEBUG environment variable:
/home/uos/Downloads/lin/libXext.so: 无法打开共享对象文件: 没有那个文件或目录
/home/uos/Downloads/lin/liblibXext.so: 无法打开共享对象文件: 没有那个文件或目录
/home/uos/Downloads/lin/libXext.so.so: 无法打开共享对象文件: 没有那个文件或目录
/home/uos/Downloads/lin/liblibXext.so.so: 无法打开共享对象文件: 没有那个文件或目录

   at CPF.Linux.XLib.XShapeCombineRegion(IntPtr display, IntPtr dest, Int32 destKind, Int32 xOff, Int32 yOff, IntPtr region, Int32 op)
   at UnoInk.X11Ink.X11InkWindow..ctor(X11Info x11Info, IntPtr mainWindowHandle)
   at UnoInk.X11Ink.X11InkProvider.Start(Window unoWindow)
```

可以尝试配置使用 libXext.so.6 版本，代码如下

```csharp
        [DllImport("libXext.so.6")]
        public static extern void XShapeCombineRegion(IntPtr display, IntPtr dest, int destKind, int xOff, int yOff, IntPtr region, int op);
```

更新之后的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8f208442ada1049cc3a5f7be789df305acb66ab4/X11/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/8f208442ada1049cc3a5f7be789df305acb66ab4/X11/DikalehebeekaJaqunicobo) 上，欢迎拉取代码阅读和构建

以上方法用于设置全应用穿透，在此基础上，支持运行过程中，动态决定穿透范围。比如在某个逻辑里面，设置 （0，0，1000，1000）的范围内支持命中，其他范围依然穿透，可以使用如下代码进行更新

```csharp
                        var xRectangle = new XRectangle()
                        {
                            x = 0,
                            y = 0, 
                            width = 1000,
                            height = 1000,
                        };
                        XRectangle* p = &xRectangle;
                        XUnionRectWithRegion(p, region, region);
                        XShapeCombineRegion(display, childWindowHandle, ShapeInput, 0, 0, region, ShapeSet);

        [DllImport(libX11)]
        public static extern int XUnionRectWithRegion(XRectangle* rectangle, IntPtr srcRegion, IntPtr destRegion);
```

更新之后的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0df132636a27746b1895b7f9a8fa347b1fe9540b/X11/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0df132636a27746b1895b7f9a8fa347b1fe9540b/X11/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0df132636a27746b1895b7f9a8fa347b1fe9540b
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0df132636a27746b1895b7f9a8fa347b1fe9540b
```

获取代码之后，进入 X11/DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

参考文档：

[如何在屏幕上显示一局部透明、鼠标点击可穿过的窗口 - V2EX](https://www.v2ex.com/t/944176 )

[2021-08-21窗口管理器杂谈 - 简书](https://www.jianshu.com/p/c49fc6c1b03e )

更多 X11 开发请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

关于在 Windows 系统下的 WPF 窗口点击穿透，请参阅 [WPF 制作支持点击穿透的高性能的透明背景异形窗口](https://blog.lindexi.com/post/WPF-%E5%88%B6%E4%BD%9C%E6%94%AF%E6%8C%81%E7%82%B9%E5%87%BB%E7%A9%BF%E9%80%8F%E7%9A%84%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84%E9%80%8F%E6%98%8E%E8%83%8C%E6%99%AF%E5%BC%82%E5%BD%A2%E7%AA%97%E5%8F%A3.html )