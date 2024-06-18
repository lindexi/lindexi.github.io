
# dotnet 在 UNO 里获取 X11 窗口指针的方法

在 UNO 的 5.2 版本，可以使用 X11 平台承载 UNO 应用。此时我需要获取到 UNO 应用的窗口的 X11 窗口指针，如此即可调用 X11 平台相关逻辑对 UNO 窗口执行一些交互

<!--more-->


<!-- CreateTime:2024/05/23 07:25:22 -->

<!-- 发布 -->
<!-- 博客 -->

本文以下的方法需要用到反射

通过阅读 UNO 的源代码，可以看到 Window 类型里面放入了不公开的 NativeWindow 属性，这个属性是平台相关的。在 X11 平台下是 Uno.WinUI.Runtime.Skia.X11.X11Window 类型。对应的 X11Window 类型的定义如下

```csharp
internal record struct X11Window(IntPtr Display, IntPtr Window, (int stencilBits, int sampleCount, IntPtr context)? glXInfo)
{
	public X11Window(IntPtr Display, IntPtr Window) : this(Display, Window, null)
	{
	}

	public readonly void Deconstruct(out IntPtr Display, out IntPtr Window, out (int stencilBits, int sampleCount, IntPtr context)? GLXInfo)
	{
		Display = this.Display;
		Window = this.Window;
		GLXInfo = this.glXInfo;
	}
}
```

获取 X11Window 里面的 Window 属性即可获取到 X11 窗口指针

反射的代码如下

```csharp
        var type = MainWindow.GetType();
        var nativeWindowPropertyInfo = type.GetProperty("NativeWindow", BindingFlags.Instance | BindingFlags.NonPublic);
        var x11Window = nativeWindowPropertyInfo!.GetMethod!.Invoke(MainWindow, null)!;
        // Uno.WinUI.Runtime.Skia.X11.X11Window
        var x11WindowType = x11Window.GetType();

        var x11WindowIntPtr = (IntPtr) x11WindowType.GetProperty("Window", BindingFlags.Instance | BindingFlags.Public)!.GetMethod!.Invoke(x11Window, null)!;

        Console.WriteLine($"Uno 窗口句柄 {x11WindowIntPtr}");
```

通过以上方式即可获取到 X11 窗口指针

但是必须说明的是，使用反射获取，也许在后续版本将会失效。我已经和 UNO 团队提需求了，请看 <https://github.com/unoplatform/uno/issues/17194>

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0f1d39d4f2bde2e60d790cb14302b5397ca0ae9c/UnoDemo/ChuchejairqaibalNallnowequyalgaw) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0f1d39d4f2bde2e60d790cb14302b5397ca0ae9c/UnoDemo/ChuchejairqaibalNallnowequyalgaw) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0f1d39d4f2bde2e60d790cb14302b5397ca0ae9c
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0f1d39d4f2bde2e60d790cb14302b5397ca0ae9c
```

获取代码之后，进入 UnoDemo/ChuchejairqaibalNallnowequyalgaw 文件夹，即可获取到源代码




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。