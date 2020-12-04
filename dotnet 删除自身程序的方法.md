# dotnet 删除自身程序的方法

本文告诉大家一个逗比方法可以用来删除程序自身

<!--more-->
<!-- CreateTime:2020/8/6 18:49:15 -->



我写了一个逗比 WPF 程序，这个程序会做邪恶的事情，会假装成小伙伴的桌面，然后小伙伴以为是桌面，接着打开任何程序都是在逗他的， 所以我期望在程序运行完成之后自动删除，这样小伙伴就不知道是谁做的

那么如何使用有趣的方法删除程序自身

可以试试下面代码

```csharp
		private static void DeleteItselfByCmd()
		{
			string command = "del /f /s /q " + Process.GetCurrentProcess().MainModule.FileName;
			Process.Start(new ProcessStartInfo("cmd.exe", "/C ping 127.1 -n 2 > nul & " + command)
			{
				WindowStyle = ProcessWindowStyle.Hidden,
				CreateNoWindow = true
			});
		}
```

在软件退出之前调用这句话，然后退出软件，此时在 cmd 执行 `ping 127.1 -n 2` 的速度不够快，因此就会等待软件退出，然后执行删除代码

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
