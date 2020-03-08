# 加强版在国内分发 UWP 应用正确方式 通过win32安装UWP应用

几乎所有国内的 UWP 开发者都知道，在国内开发 UWP 应用最大的问题不在于那么多系统的适配和不断修改的 API 接口，而是用户根本无法下载安装应用。在国内除非能掌握入口，否则想要将 UWP 开发应用在商业团队是很难的。刚好我所在的团队能将硬件设备直接给到用户，此时预装的系统里面就可以带上自主开发的 UWP 应用，解决了用户无法在应用商店安装应用的坑。但是大多数的团队都不能这样做，同时我的团队如果只是依靠硬件设备预装，那么会限制用户量。本文告诉大家如何使用传统的方法，将 UWP 作为 win32 应用安装包方法分发给用户安装

<!--more-->
<!-- CreateTime:2019/11/25 15:44:03 -->

<!-- csdn -->

将 UWP 作为安装包的方式分发也就是通过旁加载的方式，而微软干的不错的时使用旁加载如果使用的证书不清真，那么就要求用户信任证书。而大多数的用户都无法成功安装证书，我测试了几个应用，发现通过这个方式的没有一个能成功安装。所以需要解决的问题是先帮助用户安装证书，然后再安装应用。而 UWP 默认的安装程序的界面也不好看，想要定制好看的界面将需要做而外的界面开发。将 UWP 作为 win32 安装包的方式让用户安装的原理就是写一个安装程序，这个安装程序是控制台程序，在安装程序将会自动安装证书，自动安装应用。然后再写一个安装界面程序，安装界面程序可以和安装程序是两个进程，这样安装界面可以用很漂亮的 WPF 写，作出有趣的动画。因为所有 UWP 只能在 win10 运行，而 win10 系统自带 .NET 4.7 所以可以放心使用 WPF 程序。那些 WPF 程序运行不起来的系统应该是魔改的系统，这些系统也不要想 UWP 能运行

首先创建一个测试的 UWP 程序，这是一个空白的 UWP 程序，只是用来测试安装。先要求这个 UWP 程序的版本是基于 18362 的版本，当然命令行安装程序是对 UWP 版本没有要求的，只是我测试的是 18362 版本，如果小伙伴用随意的版本踩到坑了，就请自己解决

通过在 VisualStudio 右击项目，选择发布，使用旁加载方式发布，请看 [Packaging UWP apps](https://docs.microsoft.com/en-us/windows/msix/package/packaging-uwp-apps?redirectedfrom=MSDN#sideload-your-app-package) 

此时建议勾选上自动更新的选项，这样才能做到自动更新。如何做自动更新请看 [如何在国内发布 UWP 应用](https://blog.lindexi.com/post/%E5%A6%82%E4%BD%95%E5%9C%A8%E5%9B%BD%E5%86%85%E5%8F%91%E5%B8%83-UWP-%E5%BA%94%E7%94%A8.html) 和 [win10 uwp 发布旁加载自动更新](https://blog.lindexi.com/post/win10-uwp-%E5%8F%91%E5%B8%83%E6%97%81%E5%8A%A0%E8%BD%BD%E8%87%AA%E5%8A%A8%E6%9B%B4%E6%96%B0.html ) 用这两个方法可以做到旁加载的应用可以自动更新，要求 17134 最低版本

此时可以看到发布的文件夹，这里的 `*.cer` 就是证书文件，而 `*.msixbundle` 就是安装文件，在 UWP 基于不同的版本打出来的安装文件将会不同，本文只是简单告诉大家如何使用命令行安装，所以建议大家使用 18362 的版本，这样文件才会相同

在安装程序里面，拿到当前发布的文件夹的路径，然后调用 InstallApp 方法，需要注意安装程序的调试和运行都需要使用管理员权限

```csharp
        private static void InstallApp(string appFolder)
        {
            var windowsPrincipal = new WindowsPrincipal(WindowsIdentity.GetCurrent());
            if (!windowsPrincipal.IsInRole(WindowsBuiltInRole.Administrator))
            {
                Console.WriteLine("请使用管理员权限运行");
                return;
            }
        }
```

这里传入的 appFolder 就是如 `FarwheebanaHeaceababar\AppPackages\FarwheebanaHeaceababar_1.0.1.0_Debug_Test` 的文件夹，在这个文件夹里面就能找到证书文件

通过下面代码找到证书文件

```csharp
        private static string GetCerFile(string appFolder)
        {
            var cerFile = Directory.GetFiles(appFolder, "*.cer").FirstOrDefault();
            if (!File.Exists(cerFile))
            {
                Console.WriteLine("找不到 cer 证书文件");
            }

            return cerFile;
        }
```

然后调用 certutil 程序安装证书，这个程序可以在 `C:\Windows\System32\` 找到

```csharp
       private static void InstallCer(string cerFile, string appFolder)
        {
            var command = $" -addstore TrustedPeople \"{cerFile}\"";
            var processStartInfo = new ProcessStartInfo
            {
                FileName = "certutil.exe",
                Arguments = command,
                RedirectStandardOutput = true,
                Verb = "runas",
                WorkingDirectory = appFolder,
                UseShellExecute = false,
            };

            var process = new Process()
            {
                StartInfo = processStartInfo
            };
            process.Start();
            var processStandardOutput = process.StandardOutput;
            Console.WriteLine(processStandardOutput.ReadToEnd());

            process.WaitForExit();
        }
```

可以测试一下，使用命令行自己安装另一个证书

```csharp
certutil -addstore TrustedPeople C:\lindexi\foo.cer
```

然后找到安装文件

```csharp
        private static string GetBundle(string appFolder)
        {
            var bundleFile = Directory.GetFiles(appFolder, "*.msixbundle").FirstOrDefault();

            if (!File.Exists(bundleFile))
            {
                Console.WriteLine("找不到 msixbundle 安装文件");
            }

            return bundleFile;
        }
```

使用 PowerShell 的 Add-AppxPackage 安装

```csharp
       private static void InstallBundle(string bundleFile, string appFolder)
        {
            var powerShell = GetPowerShell();

            var command = $" Add-AppxPackage  -Path \"{bundleFile}\"";
            var processStartInfo = new ProcessStartInfo()
            {
                FileName = powerShell,
                Arguments = command,
                Verb = "runas",
                RedirectStandardOutput = true,
                UseShellExecute = false,
                WorkingDirectory = appFolder
            };

            var process = Process.Start(processStartInfo);
            Console.WriteLine(process.StandardOutput.ReadToEnd());
            process.WaitForExit();
        }

        private static string GetPowerShell()
        {
            return "PowerShell.exe";
        }
```

如果上面两个命令运行了，那么打开开始菜单看是否成功安装了应用，如果没有安装，那么请在命令行自己试试命令，看是不是路径没有写对

这样就完成了使用控制台安装 UWP 程序，但是这个方法存在的问题是如果用户没有开启旁加载，那么将会安装失败。如何通过命令开启旁加载请看 [win10 uwp 通过命令行脚本开启旁加载](https://blog.lindexi.com/post/win10-uwp-%E9%80%9A%E8%BF%87%E5%91%BD%E4%BB%A4%E8%A1%8C%E8%84%9A%E6%9C%AC%E5%BC%80%E5%90%AF%E6%97%81%E5%8A%A0%E8%BD%BD.html )

我就通过这个安装程序，在 NSIS 将 UWP 程序打包，在安装的时候调用安装程序，让安装程序在后台安装 UWP 应用。因为使用安装包安装可以拿到管理员权限，所以上面的安装程序不需要说明需要管理员权限

大概使用这个方法才能让用户在国内用上 UWP 应用，用上了 UWP 应用可以使用现代化的触摸和极高的渲染性能，可以作出现代的应用。但是用 UWP 的不足在于稳定性还是比较差，同时因为 UWP 的底层 API 封装没有 WPF 做的好，所以在出现稳当性问题也比较难在开发的时候找到

如果有小伙伴准备商业化使用本文的方案，欢迎告诉我，这样我好去和小伙伴吹。如果是个人开发者的话，那么自己玩就好了。因为个人开发者不需要整套的 DevOps 以及应用分发的数据传回等，如果个人开发者玩了这么多，开发量预计比应用还大

本文的命令行安装程序放在 [github](https://github.com/lindexi/lindexi_gd/tree/52d635598e2342fa74a6aaebb73ff40df0a7ad50/FarwheebanaHeaceababar) 欢迎小伙伴访问

如果不想通过命令行调用的方式安装，可以使用 win32 的方法安装，请看 [aL3891/AppxInstaller: Tools for installing Uwp apps outside the windows store](https://github.com/aL3891/AppxInstaller) 但是这个项目现在gg了，原因是微软改了接口，同时会在一些设备上安装失败

通过旁加载的应用有坑是在系统更新之后可能就无法使用了，如果小伙伴找到解决方案欢迎评论

[Install apps with the WinAppDeployCmd.exe tool - UWP apps](https://docs.microsoft.com/en-us/windows/uwp/packaging/install-universal-windows-apps-with-the-winappdeploycmd-tool)

[Create an app package with the MakeAppx.exe tool - MSIX](https://docs.microsoft.com/en-us/windows/msix/package/create-app-package-with-makeappx-tool)

[如何在国内发布 UWP 应用](https://blog.lindexi.com/post/%E5%A6%82%E4%BD%95%E5%9C%A8%E5%9B%BD%E5%86%85%E5%8F%91%E5%B8%83-UWP-%E5%BA%94%E7%94%A8.html)

[WindowsStoreAppUtils](https://github.com/apache/cordova-windows/blob/master/template/cordova/lib/WindowsStoreAppUtils.ps1)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
