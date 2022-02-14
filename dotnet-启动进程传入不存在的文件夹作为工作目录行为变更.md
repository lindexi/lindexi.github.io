
# dotnet 启动进程传入不存在的文件夹作为工作目录行为变更

本文记录在 dotnet 下，启动进程，传入不存在的文件夹作为进程的工作目录，分别在 .NET Framework 和 .NET Core 的行为

<!--more-->


<!-- CreateTime:2022/2/9 19:37:27 -->

<!-- 博客 -->
<!-- 发布 -->

在 dotnet 6 下，可以使用 ProcessStartInfo 辅助创建 Process 进程，如以下代码进行测试，传入不存在的 `Z:\Windows` 文件夹

```csharp
Console.WriteLine($"Fx {Environment.CurrentDirectory}");

if (args.Length > 0)
{
    return;
}

var location = Assembly.GetExecutingAssembly().Location;
var fileName = Path.GetFileNameWithoutExtension(location);
var directory = Path.GetDirectoryName(location);

var exe = Path.Combine(directory, fileName + ".exe");
var processStartInfo = new ProcessStartInfo(exe,"fx")
{
    WorkingDirectory = "Z:\\Windows"
};
var process = Process.Start(processStartInfo);
```

运行将会在 Process.Start 方法上抛出 System.ComponentModel.Win32Exception 说 目录名称无效

如果是在英文环境下，将会提示 `The directory name is invalid` 从而失败

但如果没有设置 ProcessStartInfo 的 WorkingDirectory 工作路径，那么默认将使用当前进程的 Environment.CurrentDirectory 值作为启动进程的工作路径

在 .NET Core 和 .NET Framework 下，启动时，设置 UseShellExecute 分别为 true 和 false 的值，行为有所不同。在不设置 ProcessStartInfo 的 WorkingDirectory 工作路径，让新的进程默认使用 Environment.CurrentDirectory 工作文件夹。但是此工作路径是一个被插拔的 U 盘的路径，如以下代码

```csharp
            Environment.CurrentDirectory = @"I:\";

            var exe = Path.Combine(directory, fileName + ".exe"); // 执行到这句代码的时候，拔出 U 盘，让 I:\ 不存在
            var processStartInfo = new ProcessStartInfo(exe, "fx")
            {
                UseShellExecute = true, // 也设置为 false 的值
            };
            var process = Process.Start(processStartInfo);
            process.WaitForExit();
```

我使用 .NET 6 和 .NET Framework 4.5 进行分别的测试，测试如下：

在 .NET Core 下，设置 UseShellExecute=false 的值，运行结果是：成功，新进程工作路径等于 `I:\` 路径

在 .NET Core 下，设置 UseShellExecute=true 的值，运行结果是：成功，新进程工作路径等于 `C:\Windows` 路径

在 .NET Framework 下，设置 UseShellExecute=false 的值，运行结果是：运行 Process.Start 失败，提示 `System.ComponentModel.Win32Exception: '目录名称无效。'` 错误

在 .NET Framework 下，设置 UseShellExecute=true 的值，运行结果是：成功，新进程工作路径等于 `C:\Windows` 路径








<!-- 这是在 dotnet core 上的行为。 在 .NET Framework 下，以上代码不会抛出任何异常，且新开的进程拿到的工作路径是 "C:\Windows" 文件夹

- 在 .NET Core 下，传入 ProcessStartInfo 的 WorkingDirectory 工作路径是不存在的文件夹，将抛出异常
- 在 .NET Framework 下，传入不存在的文件夹，能正常开启进程，且新进程的工作路径是 "C:\Windows" 文件夹

另外有一个例外的行为是，如果此时的 Environment.CurrentDirectory 的文件夹是一个不存在的文件夹，例如原本是指向 U 盘，但是在启动进程时，被拔出 U 盘，那么此时没有什么事情发生。但行为依然有以下的不同

- 在 .NET Core 下，传入 ProcessStartInfo 的 WorkingDirectory 工作路径是空，且 Environment.CurrentDirectory 的文件夹是一个不存在的文件夹。能启动新进程，且新进程的工作路径和当前进程的 Environment.CurrentDirectory 相同
- 在 .NET Framework 下，能正常开启进程，且新进程的工作路径是 "C:\Windows" 文件夹 -->

<!-- 

根据 UseShellExecute 参数决定采用哪个方式启动

```csharp
        private bool StartCore(ProcessStartInfo startInfo)
        {
            return startInfo.UseShellExecute
                ? StartWithShellExecuteEx(startInfo)
                : StartWithCreateProcess(startInfo);
        }
```

在 .NET Core 下，创建进程的代码是通过如下方式

```csharp
                Interop.Shell32.SHELLEXECUTEINFO shellExecuteInfo = new Interop.Shell32.SHELLEXECUTEINFO()
                {
                    cbSize = (uint)sizeof(Interop.Shell32.SHELLEXECUTEINFO),
                    lpFile = fileName,
                    lpVerb = verb,
                    lpParameters = parameters,
                    lpDirectory = directory,
                    fMask = Interop.Shell32.SEE_MASK_NOCLOSEPROCESS | Interop.Shell32.SEE_MASK_FLAG_DDEWAIT
                };

                if (startInfo.ErrorDialog)
                    shellExecuteInfo.hwnd = startInfo.ErrorDialogParentHandle;
                else
                    shellExecuteInfo.fMask |= Interop.Shell32.SEE_MASK_FLAG_NO_UI;

                shellExecuteInfo.nShow = startInfo.WindowStyle switch
                {
                    ProcessWindowStyle.Hidden => Interop.Shell32.SW_HIDE,
                    ProcessWindowStyle.Minimized => Interop.Shell32.SW_SHOWMINIMIZED,
                    ProcessWindowStyle.Maximized => Interop.Shell32.SW_SHOWMAXIMIZED,
                    _ => Interop.Shell32.SW_SHOWNORMAL,
                };
                ShellExecuteHelper executeHelper = new ShellExecuteHelper(&shellExecuteInfo);
                if (!executeHelper.ShellExecuteOnSTAThread())
                {
                    // 忽略代码
                }
```

在 .NET Framework 下的代码如下

```csharp
              process = Microsoft.Win32.NativeMethods.CreateProcess((string) null, stringBuilder, (Microsoft.Win32.NativeMethods.SECURITY_ATTRIBUTES) null, (Microsoft.Win32.NativeMethods.SECURITY_ATTRIBUTES) null, true, num1, num2, lpCurrentDirectory, lpStartupInfo, lpProcessInformation);
```

可以看到底层创建的方法不相同。不过底层的 ShellExecute 也是会调用到 CreateProcess 方法的 -->

更多请看 [c# - Win32Exception: The directory name is invalid - Stack Overflow](https://stackoverflow.com/questions/990562/win32exception-the-directory-name-is-invalid )

[CreateProcess and ShellExecute differences - Stack Overflow](https://stackoverflow.com/q/10747479/6116637 )

本文所有代码放在[github](https://github.com/lindexi/lindexi_gd/tree/f7696a3e9f33dfcbfdd8ab92afaa77ab668dfeb9/HebarlawkuKekebuwagay) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f7696a3e9f33dfcbfdd8ab92afaa77ab668dfeb9/HebarlawkuKekebuwagay) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f7696a3e9f33dfcbfdd8ab92afaa77ab668dfeb9
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 HebarlawkuKekebuwagay 文件夹





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。