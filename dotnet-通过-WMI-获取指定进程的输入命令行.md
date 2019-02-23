
# dotnet 通过 WMI 获取指定进程的输入命令行

本文告诉大家如何使用 WMI 通过 Process 获取这个进程传入的命令行

<!--more-->


<!-- csdn -->
<!-- 标签：dotnet,C#,WMI -->

使用下面代码，使用 Win32_Process 拿到所有的进程，通过 WHERE 判断当前的进程，然后拿到进程传入的命令

```csharp
private static string GetCommandLine(this Process process)
{
    using (ManagementObjectSearcher searcher = new ManagementObjectSearcher("SELECT CommandLine FROM Win32_Process WHERE ProcessId = " + process.Id))
    using (ManagementObjectCollection objects = searcher.Get())
    {
        return objects.Cast<ManagementBaseObject>().SingleOrDefault()?["CommandLine"]?.ToString();
    }

}
```

获取所有的进程的命令行参数

```csharp
private static void Main()
{
    foreach (var process in Process.GetProcesses())
    {
        try
        {
            Console.WriteLine(process.GetCommandLine());
        }
        catch (Win32Exception ex) when ((uint)ex.ErrorCode == 0x80004005)
        {
            // Intentionally empty - no security access to the process.
        }
        catch (InvalidOperationException)
        {
            // Intentionally empty - the process exited before getting details.
        }

    }
}
```

[https://stackoverflow.com/a/2633674/6116637](https://stackoverflow.com/a/2633674/6116637 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。