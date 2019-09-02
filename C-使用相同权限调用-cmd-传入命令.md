
# C# 使用相同权限调用 cmd 传入命令

本文告诉大家如何使用相同权限调用cmd并且传入命令。

<!--more-->


<!-- 不发布 -->

如果想要用相同的权限运行一个程序，可以使用 ProcessStartInfo 的方法

```csharp
            var processStartInfo = new ProcessStartInfo()
            {
                Verb = "runas", // 如果程序是管理员权限，那么运行 cmd 也是管理员权限
                FileName = "cmd.exe",
            };
```

只需要设置 `Verb = "runas"` 就可以使用相同的权限运行程序。

如何设置程序使用管理员权限运行，请看

所以需要修改一下在 C# 调用 ProcessStartInfo 使用 cmd 并且传入参数的方法

```csharp
            var processStartInfo = new ProcessStartInfo()
            {
                Verb = "runas", // 如果程序是管理员权限，那么运行 cmd 也是管理员权限
                FileName = "cmd.exe",
                UseShellExecute = false,
                RedirectStandardInput = true,
                RedirectStandardOutput = true,
                RedirectStandardError = true,
                CreateNoWindow = false, // 如果需要隐藏窗口，设置为 true 就不显示窗口
                StandardOutputEncoding = Encoding.UTF8,
                Arguments = "/K " + str + " &exit",
            };

            var p = Process.Start(processStartInfo);
```

这里传入的 Arguments 需要使用 `/K` 或 `/C` 放在最前，不然 cmd 不会执行参数。

如果需要拿到输出就需要用到其他的代码，所有的代码请看下面，代码可以直接复制使用。

```csharp
       private static (string output, int exitCode) Control(string str)
        {
            var processStartInfo = new ProcessStartInfo()
            {
                Verb = "runas", // 如果程序是管理员权限，那么运行 cmd 也是管理员权限
                FileName = "cmd.exe",
                UseShellExecute = false,
                RedirectStandardInput = true,
                RedirectStandardOutput = true,
                RedirectStandardError = true,
                CreateNoWindow = false, // 如果需要隐藏窗口，设置为 true 就不显示窗口
                StandardOutputEncoding = Encoding.UTF8,
                Arguments = "/K " + str + " &exit",
            };

            var p = Process.Start(processStartInfo);

            //向cmd窗口发送输入信息

            p.StandardInput.AutoFlush = true;
            //向标准输入写入要执行的命令。这里使用&是批处理命令的符号，表示前面一个命令不管是否执行成功都执行后面(exit)命令，如果不执行exit命令，后面调用ReadToEnd()方法会假死
            //同类的符号还有&&和||前者表示必须前一个命令执行成功才会执行后面的命令，后者表示必须前一个命令执行失败才会执行后面的命令


            //获取cmd窗口的输出信息
            var output = "";

            var task = p.StandardOutput.ReadToEndAsync();

            task.Wait(2000);

            if (task.IsCompleted)
            {
                output += task.Result;
            }

            task = p.StandardError.ReadToEndAsync();

            task.Wait(2000);

            if (task.IsCompleted)
            {
                output += task.Result;
            }

            Console.WriteLine(output);

            p.WaitForExit(10000); //等待程序执行完退出进程
            p.Close();
            int ec = 0;
            try
            {
                ec = p.ExitCode;
            }
            catch (Exception)
            {
            }

            return (output + "\r\n", ec);
        }
```

那么需要降权运行某个程序怎么办？

可以使用 `Process.Start("explorer")` 然后参数传入 `xx.exe` 运行，另外执行时还存在另一个问题，如果发现双击可以运行程序，但是调用 `Process.Start` 无法启动程序，请看下面的博客

[Process执行路径 - haungtengxiao](https://xinyuehtx.github.io/post/Process%E6%89%A7%E8%A1%8C%E8%B7%AF%E5%BE%84.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。