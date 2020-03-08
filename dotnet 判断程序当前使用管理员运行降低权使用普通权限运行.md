# dotnet 判断程序当前使用管理员运行降低权使用普通权限运行

有一些程序是不想通过管理员权限运行的，因为在很多文件的读写，如果用了管理员权限程序写入的程序，其他普通权限的程序是无法直接访问的。

本文告诉大家如何判断当前的程序是通过管理员权限运行，然后通过资源管理器使用普通权限运行

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


通过下面代码可以判断当前的程序是管理员权限运行

```csharp
            var identity = WindowsIdentity.GetCurrent();
            var principal = new WindowsPrincipal(identity);
            if (principal.IsInRole(WindowsBuiltInRole.Administrator))
            {
                // 当前正在以管理员权限运行。
            }
```

如果是 dotnet core 程序，需要安装 [Microsoft.Windows.Compatibility](https://www.nuget.org/packages/Microsoft.Windows.Compatibility) 才可以使用上面代码

通过 Explorer 运行自己，在 dotnet framework 程序和 dotnet core 程序在获得自己的 exe 文件的方法是不同的

在 dotnet framework 程序可以直接在 Main 函数通过 Assembly.GetEntryAssembly().Location 拿到 exe 文件的路径

```csharp
                Process.Start("explorer.exe", Assembly.GetEntryAssembly().Location);

```

但是如果在 dotnet core 程序，通过 Assembly.GetEntryAssembly().Location 会拿到 xx.dll 而不是 exe 的路径，需要使用下面的代码拿到 exe 的文件

```csharp
// 方法1

                var file = new FileInfo(Assembly.GetExecutingAssembly().Location);
                var exe = Path.Combine(file.DirectoryName, file.Name.Replace(file.Extension, "")+".exe");

// 方法2
                var exe = Process.GetCurrentProcess().MainModule.FileName;

// 更多方法
```

然后自己关闭

```csharp
            var identity = WindowsIdentity.GetCurrent();
            var principal = new WindowsPrincipal(identity);
            if (principal.IsInRole(WindowsBuiltInRole.Administrator))
            {
                var file = new FileInfo(Assembly.GetExecutingAssembly().Location);
                var exe = Path.Combine(file.DirectoryName, file.Name.Replace(file.Extension, "") + ".exe");
           	
                // 检测到当前进程是以管理员权限运行的，于是降权启动自己之后，把自己关掉。
                Process.Start("explorer.exe", Assembly.GetEntryAssembly().Location);
                Environment.Exit(0);
            }
```

[在 Windows 系统上降低 UAC 权限运行程序（从管理员权限降权到普通用户权限） - walterlv](https://walterlv.com/post/start-process-with-lowered-uac-privileges.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
