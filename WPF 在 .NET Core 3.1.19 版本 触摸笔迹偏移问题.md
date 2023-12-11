# WPF 在 .NET Core 3.1.19 版本 触摸笔迹偏移问题

在更新到 .NET 6 发布之前的，在 2021.11.02 的 .NET Core 版本，都会存在此问题。在 WPF 应用里面，如果在高 DPI 下，进行触摸书写，此时的笔迹将会偏移。核心原因是在这几个版本的 WPF 使用 module initializer 代替原有的 IL 注入，但是代码有锅，导致初始化的逻辑没有正确初始化

<!--more-->
<!-- CreateTime:2021/11/2 9:38:11 -->


<!-- 发布 -->

最佳解决方法：升级 .NET Core 版本即可，最新 .NET Runtime 可解

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

此问题已修，但是需要等待 .NET 6 发布才同步更新到各个版本，包括 .NET Core 3.1 的版本。现在已经全部 .NET Core 版本都修复了，只需要更新运行时即可

更多请看 [WPF 在 .NET Core 3.1.19 版本没有跟随 DPI 缩放文本过小问题](https://blog.lindexi.com/post/WPF-%E5%9C%A8-.NET-Core-3.1.19-%E7%89%88%E6%9C%AC%E6%B2%A1%E6%9C%89%E8%B7%9F%E9%9A%8F-DPI-%E7%BC%A9%E6%94%BE%E6%96%87%E6%9C%AC%E8%BF%87%E5%B0%8F%E9%97%AE%E9%A2%98.html )
