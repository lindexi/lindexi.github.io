# dotnet 判断其他进程未响应

在 dotnet 有很多方法判断其他软件的其他进程窗口未响应

<!--more-->
<!-- CreateTime:2019/12/17 19:48:40 -->

<!-- 发布 -->

最简单方法是通过 Process 判断

```csharp
process.Responding
```

如判断资源管理器是否未响应

```csharp
            Process.Start("explorer.exe");

            foreach (var process in Process.GetProcesses())
            {
                if (process.ProcessName == "explorer")
                {
                    if (!process.Responding)
                    {
                        Console.WriteLine("资源管理器没有响应");
                        Console.Read();
                        return;
                    }
                }
            }
```

另一个方法是通过 SendMessageTimeout 方法，这个方法将指定的消息发送到一个或多个窗口，因为窗口未响应的原因是没有处理消息。尝试发送一个消息，如果没有发送到对应的窗口，那么就是窗口未响应

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
