
# WPF 在 .NET Core 3.1.19 版本没有跟随 DPI 缩放文本过小问题

本文告诉大家一个坑，在 .NET Core 3.1.19 版本，因为 WPF 框架的处理不当，而让应用没有感知 DPI 而不会跟随缩放，让文本过小的问题。本文将告诉大家解决方法和原因

<!--more-->


<!-- CreateTime:2021/10/11 8:55:26 -->

<!-- 发布 -->

最佳解决方法：升级 .NET Core 版本即可

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

特别感谢 [ChristophHornung](https://github.com/ChristophHornung) 提供的如上方法

核心原因是因为 [这段代码](https://github.com/dotnet/wpf/blob/main/src/Microsoft.DotNet.Wpf/src/DirectWriteForwarder/main.cpp#L172-L206) 执行时机问题，没有足够早执行。为什么会有这个问题？原因是在更换 module initializer 进行模块初始化的锅，对这么大的框架来说，任何的更改也许都在挖坑

详细请参阅如下内容：

- [[release/3.1] Application scaling regression. Revert ildasm/ilasm tools to 4.6.2 to fix module initializer injection regression. by ryalanms · Pull Request #5377 · dotnet/wpf](https://github.com/dotnet/wpf/pull/5377 )
- [Application not scaled with .NET Core 3.1.19 · Issue #5375 · dotnet/wpf](https://github.com/dotnet/wpf/issues/5375 )
- [Application window is now small with very small text · Issue #5472 · dotnet/wpf](https://github.com/dotnet/wpf/issues/5472 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。