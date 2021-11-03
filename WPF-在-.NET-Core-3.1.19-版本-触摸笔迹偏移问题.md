
# WPF 在 .NET Core 3.1.19 版本 触摸笔迹偏移问题

在更新到 .NET 6 发布之前的，在 2021.11.02 的 .NET Core 版本，都会存在此问题。在 WPF 应用里面，如果在高 DPI 下，进行触摸书写，此时的笔迹将会偏移。核心原因是在这几个版本的 WPF 使用 module initializer 代替原有的 IL 注入，但是代码有锅，导致初始化的逻辑没有正确初始化

<!--more-->



<!-- 发布 -->

最佳解决方法：升级 .NET Core 版本即可（等待 .NET 6 发布之后）

其次的解决方法是在 App 的启动方法添加如下代码

```csharp
public partial class App : Application
{
	protected override void OnStartup(StartupEventArgs e)
	{
		SetProcessDPIAware();
		base.OnStartup(e);
	}

	[DllImport("user32.dll", SetLastError = true)]
	private static extern bool SetProcessDPIAware();
}
```

加应用程序清单也可以，加完之后，记得在 VisualStudio 重新生成

特别感谢 [ChristophHornung](https://github.com/ChristophHornung) 提供的如上方法

此问题已修，但是需要等待 .NET 6 发布才同步更新到各个版本，包括 .NET Core 3.1 的版本

更多请看 [WPF 在 .NET Core 3.1.19 版本没有跟随 DPI 缩放文本过小问题](https://blog.lindexi.com/post/WPF-%E5%9C%A8-.NET-Core-3.1.19-%E7%89%88%E6%9C%AC%E6%B2%A1%E6%9C%89%E8%B7%9F%E9%9A%8F-DPI-%E7%BC%A9%E6%94%BE%E6%96%87%E6%9C%AC%E8%BF%87%E5%B0%8F%E9%97%AE%E9%A2%98.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。