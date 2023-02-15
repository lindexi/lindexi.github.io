
# dotnet 已知问题 使用 Directory.EnumerateXXX 方法枚举 C 盘根路径可能错误的问题

在 dotnet 里面，可以使用 Directory.EnumerateXXX 系列方法进行枚举文件或文件夹。在准备枚举驱动器根路径的文件或文件夹时，可能获取到错误的路径。错误的步骤在于传入的是如 C: 不带斜杠的路径，且存在同驱动器磁盘下的非根路径工作路径

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

特别感谢 [神樹桜乃](https://github.com/KodamaSakuno) 和 [若凡](https://github.com/zhuxb711) 两位大佬，让我明白了此问题和原因

此问题已报告给 dotnet 官方，详细请看 [https://github.com/dotnet/runtime/issues/82085](https://github.com/dotnet/runtime/issues/82085)

这是一个稍微复杂的问题，大部分情况下大家不会遇到。这是一个从 .NET Core 3.1 到 .NET 7 都存在的问题，且基本上和系统环境无关

在 dotnet 里面可以使用 Directory.EnumerateXXX 系列方法进行枚举文件或文件夹，比如 Directory.EnumerateFiles 和 Directory.EnumerateFileSystemEntries 等方法。在使用这些方法进行枚举驱动器的根路径，如 `C:` 盘的文件或文件夹时，如果传入的路径参数是不带斜杠的，如 `C:` 而不是 `C:\` 时，如果同时此时的工作路径是同驱动器下的非根路径，如 `C:\lindexi` 文件夹时，将会枚举出错误的路径

以下是 [神樹桜乃](https://github.com/KodamaSakuno) 大佬给出的最简复现步骤

先在 C 盘创建一个名为 `bug` 的文件夹，在此文件夹里面放入 `a` `b` `c` 三个文件

接着设置工作路径为 `C:\bug` 且运行以下代码

```csharp
foreach (var item in Directory.EnumerateFiles("C:"))
{
    Console.WriteLine(item);
}
```

此时的输出居然是如下代码

```csharp
C:\a
C:\b
C:\c
```

也就是中间的 bug 文件夹没有输出，输出的是错误的路径。预期的输出应该如下

```csharp
C:\bug\a
C:\bug\b
C:\bug\c
```

这个已知问题的核心原因是在 dotnet 底层里面调用的 [GetFullPathNameW](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-getfullpathnamew ) 函数行为和 dotnet 预期的不相同，如 [官方文档](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-getfullpathnamew ) 描述如下内容

> If you specify "U:" the path returned is the current directory on the "U:\" drive

也就是如果传入参数是 `C:` 不带斜杠的，将会使用传入的驱动器的当前工作路径。这也就导致了工作路径是同驱动器下的非根路径将会枚举到工作路径下的文件或文件夹，也就是上面例子里可以枚举到 a 和 b 和 c 文件的原因。只不过在 dotnet 的对外输出层拼接路径时，依然用的是传入的 `C:` 进行拼接，从而导致了输出错误

有一个可以用来实验的步骤是：先在 C 盘创建 lindexi 文件夹，打开 cmd 进入 `C:\lindexi` 文件夹，再使用 `dotnet run` 命令运行测试的项目，在测试的项目里面运行 `Console.WriteLine(Path.GetFullPath("C:"));` 这句代码。可以看到输出的是 `C:\lindexi` 文件夹，证明了 Windows 的底层行为符合文档

如何规避此问题？只需要破坏其条件即可，也就是在传入路径参数加上斜杠后缀，如 `C:\` 路径，即可解决此问题




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。