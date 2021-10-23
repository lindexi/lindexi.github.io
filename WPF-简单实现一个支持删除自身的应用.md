
# WPF 简单实现一个支持删除自身的应用

我准备写一个逗比的应用，然而我担心被小伙伴看到这个应用的文件从而知道是我写的，于是我就需要实现让应用能自删除的功能。核心实现方法就是调用 cmd 传入命令行，等待几秒之后删除文件

<!--more-->


<!-- 发布 -->


应用程序在运行时，是不能将 exe 文件进行删除的。但是可以将 exe 改名以及在驱动器内进行移动文件

删除应用程序可以让 cmd 进行删除，在 cmd 可以使用 timeout 命令延迟，然后通过 `&&` 进行执行后续逻辑，从而实现延迟执行命令。让 cmd 延迟执行 DEL 命令进行删除应用，在应用调用删除之后，让应用程序结束即可
 
代码如下

```csharp
        static void Main(string[] args)
        {
            var fileName = Process.GetCurrentProcess().MainModule.FileName;
            DelayDeleteFile(fileName, 2);
        }

        private static void DelayDeleteFile(string fileName, int delaySecond = 2)
        {
            fileName = Path.GetFullPath(fileName);
            var folder = Path.GetDirectoryName(fileName);
            var currentProcessFileName = Path.GetFileName(fileName);

            var arguments = $"/c timeout /t {delaySecond} && DEL /f {currentProcessFileName} ";

            var processStartInfo = new ProcessStartInfo()
            {
                Verb = "runas", // 如果程序是管理员权限，那么运行 cmd 也是管理员权限
                FileName = "cmd",
                UseShellExecute = false,
                CreateNoWindow = true, // 如果需要隐藏窗口，设置为 true 就不显示窗口
                Arguments = arguments,
                WorkingDirectory = folder,
            };

            Process.Start(processStartInfo);
        }
```


本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/62aeb3d73ca3bf97f24a7283a61bce8b7774e799/QarnafahayWalllukerrairbar) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/62aeb3d73ca3bf97f24a7283a61bce8b7774e799/QarnafahayWalllukerrairbar) 欢迎访问

可以通过如下方式获取本文代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 62aeb3d73ca3bf97f24a7283a61bce8b7774e799
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 QarnafahayWalllukerrairbar 文件夹





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。