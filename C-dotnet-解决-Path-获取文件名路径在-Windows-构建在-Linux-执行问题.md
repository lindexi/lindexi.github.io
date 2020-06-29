
# C# dotnet 解决 Path 获取文件名路径在 Windows 构建在 Linux 执行问题

最近需要支持国产的 UOS 系统，这个系统我采用了 Xamarin 加上 GTK 开发，而我的日志系统有一个功能是记录日志的时候传入当前的文件路径，此时 NuGet 包是在 Windows 下构建的，因此传入的路径是 Window 构建服务器路径。此时在 Linux 上尝试获取文件名就炸了

<!--more-->


<!-- 发布 -->

在 dotnet 中，可以通过 CallerFilePath 特性获取调用当前方法的文件路径是哪个，请看下面代码

```csharp
public void DoProcessing()
{
    TraceMessage("Something happened.");
}

public void TraceMessage(string message,
        [System.Runtime.CompilerServices.CallerMemberName] string memberName = "",
        [System.Runtime.CompilerServices.CallerFilePath] string sourceFilePath = "",
        [System.Runtime.CompilerServices.CallerLineNumber] int sourceLineNumber = 0)
{
    System.Diagnostics.Trace.WriteLine("message: " + message);
    System.Diagnostics.Trace.WriteLine("member name: " + memberName);
    System.Diagnostics.Trace.WriteLine("source file path: " + sourceFilePath);
    System.Diagnostics.Trace.WriteLine("source line number: " + sourceLineNumber);
}

// Sample Output:
//  message: Something happened.
//  member name: DoProcessing
//  source file path: c:\Users\username\Documents\Visual Studio 2020\Projects\CallerInfoCS\CallerInfoCS\Form1.cs
//  source line number: 31
```

上面代码是官方的例子，这是在构建的时候，自动注入的值。因此在 Windows 服务器构建的 NuGet 包给日志注入字符串是构建服务器上的代码文件的路径

而我记日志只是使用了文件名而已，如下面代码

```csharp
        public static void LogInfo(this ILogger logger, string message,
            string traceId = null,
            string userId = null,
            [System.Runtime.CompilerServices.CallerMemberName]
            string memberName = "",
            [System.Runtime.CompilerServices.CallerFilePath]
            string sourceFilePath = "",
            [System.Runtime.CompilerServices.CallerLineNumber]
            int sourceLineNumber = 0,
            params string[] tags)
        {
            // 刚好在 Linux 下构建的在 Linux 下运行，而在 Windows 构建的库在 Windows 下执行。此时使用 GetFileNameWithoutExtension 能保持输入路径和解析相同
            // 假定在 Windows 下构建而在 Linux 下构建，只是让路径变长而已，我相信咱的日志系统炸不了…… 或者说，炸了再说
            // 炸了的解决方法是在 dotnet runtime\src\libraries\System.Private.CoreLib\src\System\IO\Path.cs 的 GetFileName 方法里面将 `PathInternal.IsDirectorySeparator(path[i])` 替换为实际需要的 \ 或 / 符号
            var classFile = Path.GetFileNameWithoutExtension(sourceFilePath);

            // 忽略代码
        }
```

如注释，使用 Path.GetFileNameWithoutExtension 如果在 Linux 运行，传入的是 Windows 下的路径，那么此时将拿不到文件名

原因是 Path.GetFileNameWithoutExtension 的实现如下

```csharp
        public static ReadOnlySpan<char> GetFileNameWithoutExtension(ReadOnlySpan<char> path)
        {
            ReadOnlySpan<char> fileName = GetFileName(path);
            int lastPeriod = fileName.LastIndexOf('.');
            return lastPeriod == -1 ?
                fileName : // No extension was found
                fileName.Slice(0, lastPeriod);
        }

        public static ReadOnlySpan<char> GetFileName(ReadOnlySpan<char> path)
        {
            int root = GetPathRoot(path).Length;

            // We don't want to cut off "C:\file.txt:stream" (i.e. should be "file.txt:stream")
            // but we *do* want "C:Foo" => "Foo". This necessitates checking for the root.

            for (int i = path.Length; --i >= 0;)
            {
                if (i < root || PathInternal.IsDirectorySeparator(path[i]))
                    return path.Slice(i + 1, path.Length - i - 1);
            }

            return path;
        }
```

上面代码的 GetFileName 的 `PathInternal.IsDirectorySeparator` 判断就不对了，因此就拿不到文件路径了。因为在 Windows 下添加的字符串默认使用的是的是 `\` 分割，因此 Linux 下无法解析。反过来，如果是在 Linux 服务器构建的，此时在 Windows 下解析没有问题，因为在 Windows 下的 IsDirectorySeparator 方法的判断是两个方向的都可以

解决方法就是自己抄 `dotnet runtime\src\libraries\System.Private.CoreLib\src\System\IO\Path.cs` 的 GetFileNameWithoutExtension 方法，修改判断文件夹符号

这是我修改的代码

```csharp
        public static ReadOnlySpan<char> GetFileNameWithoutExtension(ReadOnlySpan<char> path)
        {
            ReadOnlySpan<char> fileName = GetFileName(path);
            int lastPeriod = fileName.LastIndexOf('.');
            return lastPeriod == -1 ?
                fileName : // No extension was found
                fileName.Slice(0, lastPeriod);
        }

        public static ReadOnlySpan<char> GetFileName(ReadOnlySpan<char> path)
        {
            int root = GetPathRoot(path).Length;

            // We don't want to cut off "C:\file.txt:stream" (i.e. should be "file.txt:stream")
            // but we *do* want "C:Foo" => "Foo". This necessitates checking for the root.

            for (int i = path.Length; --i >= 0;)
            {
                if (i < root || path[i] == '/' || path[i] == '\\')
                    return path.Slice(i + 1, path.Length - i - 1);
            }

            return path;
        }
```

特别吐槽 Linux 系统的文件路径的坑，在这里被坑了几次了

更好的解决方法是在 Linux 运行的内容就在 Linux 构建，本来我的主项目也是在 Linux 构建的。不过这里炸的是 CBB 公共组件，这些 NuGet 是在 Window 构建的，因为每个 NuGet 都支持 .NET Framework 和 .NET Core 版本

尽管我不能吐槽 UOS 系统，客观说，这个系统现在还是不开源的。我没有看到这个系统的优势。也许唯一的优势就是咱国家的有政策在推动，加上底子是深度的

我依然还会努力去支持国产的系统，尽管支持会让我加好多班，业余时间也用在支持上。整个 dotnet 都是开源的，而且还是完全开源的，加上现在龙芯等在硬件层面对 .NET 添加了运行时优化，此时在 UOS 上基于 .NET 构建从最低到最顶的完全自主研发的软件是十分简单的事情

这里需要说明一下的是完全开源的 .NET 完全允许贴牌，也就是改个名字，说这是自己完全拥有的技术，这是没有问题的，官方也是支持的。如果在贴牌之后还能将自己发现的 Bug 等提到官方，那么官方就更是推荐这样的行为。使用 .NET 的优势在于不会被卡，因为从编译到运行时到库都是完全开源，同时可自行构建，无版权问题。假设现在 M$ 断绝所有技术支持，包括抹掉了中国微软，此时对 .NET 也多少影响，除了可能存在的新代码无法拿到外，现有的代码同时基于现有代码继续开发没有任何问题。咱能改代码，也改的动





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。