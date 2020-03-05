# WPF Process.Start 出现 Win32Exception 异常

我在使用 WPF 调用其他软件的时候，发现出现Win32Exception异常。

<!--more-->
<!-- CreateTime:2019/12/8 15:11:56 -->

<!-- csdn -->
<div id="toc"></div>

如果使用下面的代码启动另一个软件，那么在启动的软件路径不存在时，就会出现异常`System.ComponentModel.Win32Exception` 没有其他信息

```csharp
	        var st = new ProcessStartInfo(ProcessName,
                "-p " + Port);

            if (NeedHidden)
            {
                st.CreateNoWindow = true;
                st.WindowStyle = ProcessWindowStyle.Hidden;
            }

            var remoteGuardian = Process.Start(st); //监控远程应用
```

上面的代码的 NeedHidden 表示是否需要隐藏窗口，如果设置为 false ，启动控制台会出现黑窗。代码 ProcessName 就是其他的软件的路径。

如果这时 File.Exists(ProcessName) 为 false ，那么 Process.Start 就出现`System.ComponentModel.Win32Exception`异常，而且这个异常没有其他的信息

所以建议在调用 Process.Start 之前判断路径是否存在

```csharp
	        if (!File.Exists(ProcessName))
            {
                throw new ArgumentException("启动软件路径不存在" + ProcessName);
            }
```

为什么会出现找不到路径？我使用的是[WPF 封装 dotnet remoting 调用其他进程](https://blog.lindexi.com/post/WPF-%E5%B0%81%E8%A3%85-dotnet-remoting-%E8%B0%83%E7%94%A8%E5%85%B6%E4%BB%96%E8%BF%9B%E7%A8%8B.html )里面需要引用一个库用来做远程的软件，因为我使用一个库 A 引用了远程的软件，使用程序B引用了A，因为没有直接引用远程软件，所以就没有在输入路径找到这个文件，所以出现异常。

其他可能出现异常的是：

 - 启动的程序弹出 UAC 被用户取消

更多关于 remoting 的博客

[WPF 封装 dotnet remoting 调用其他进程](https://blog.lindexi.com/post/WPF-%E5%B0%81%E8%A3%85-dotnet-remoting-%E8%B0%83%E7%94%A8%E5%85%B6%E4%BB%96%E8%BF%9B%E7%A8%8B.html )

[.net remoting 抛出异常](https://lindexi.gitee.io/post/.net-remoting-%E6%8A%9B%E5%87%BA%E5%BC%82%E5%B8%B8.html )

[.net remoting 使用事件](https://lindexi.gitee.io/post/.net-remoting-%E4%BD%BF%E7%94%A8%E4%BA%8B%E4%BB%B6.html )

![](https://i.loli.net/2018/04/08/5aca001656bdd.jpg)

感谢

[walterlv](https://walterlv.github.io/) 告诉我另一个出现异常的情况

[Miss_Bread](https://blog.csdn.net/miss_bread ) 提供的图片

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
