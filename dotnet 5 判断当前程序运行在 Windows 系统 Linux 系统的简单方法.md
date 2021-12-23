# dotnet 5 判断当前程序运行在 Windows 系统 Linux 系统的简单方法

本文告诉大家使用 dotnet 5 提供的 System.OperatingSystem 类的方法进行快速且简单判断当前程序所运行在的系统

<!--more-->
<!-- 发布 -->

判断系统的简单代码示例：

```csharp
            if (System.OperatingSystem.IsWindows())
            {
                if (System.OperatingSystem.IsWindowsVersionAtLeast(10, 0, 19043))
                {

                }
            }
            else if (System.OperatingSystem.IsLinux())
            {

            }
            else if (System.OperatingSystem.IsMacOS())
            {

            }
            else if (System.OperatingSystem.IsIOS())
            {

            }
            else if (System.OperatingSystem.IsAndroid())
            {

            }
            else if (System.OperatingSystem.IsBrowser())
            {

            }
            else if (System.OperatingSystem.IsTvOS())
            {

            }
            else if (System.OperatingSystem.IsFreeBSD())
            {

            }
```

可以很方便通过 IsXx 的方式判断当前是运行在哪个系统上。可以通过 IsXxVersionAtLeast 这一组方法判断是否当前运行的系统版本大于等于给定的版本

通过这些功能，可以很方便编写特定功能的代码

在 dotnet 5 之前，需要通过 RuntimeInformation.IsOSPlatform 方法进行判断系统，代码如下

```csharp
            if (RuntimeInformation.IsOSPlatform(OSPlatform.Windows))
            {
                
            }
```

更多请看 [OperatingSystem Class (System) Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/api/system.operatingsystem?view=net-6.0 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
