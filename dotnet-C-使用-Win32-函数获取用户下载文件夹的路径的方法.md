
# dotnet C# 使用 Win32 函数获取用户下载文件夹的路径的方法

大家都知道，在 dotnet 里面的可以使用 Environment.GetFolderPath 方法配合 Environment.SpecialFolder 枚举列出当前运行环境下的一些特殊文件夹。然而 SpecialFolder 枚举不包含对 Download 下载文件夹的枚举值，如咱需要获取用户当前的下载文件夹，需要使用 Win32 方法来辅助获取

<!--more-->


<!-- CreateTime:2024/04/13 07:05:11 -->

<!-- 发布 -->
<!-- 博客 -->

在 dotnet 官方，已经有人提议给 SpecialFolder 添加更多的枚举，相应的提议请参阅： <https://github.com/dotnet/runtime/issues/70484>

在 Windows 上，核心实现是通过 SHGetKnownFolderPath 这个 Win32 方法，先查阅文档，定义下载文件的 Guid 值和此 Win32 方法，代码如下

```csharp
var downloadFolderGuid = new Guid("374DE290-123F-4565-9164-39C4925E467B");

[DllImport("shell32.dll", CharSet = CharSet.Unicode, ExactSpelling = true, PreserveSig = false)]
static extern string SHGetKnownFolderPath([MarshalAs(UnmanagedType.LPStruct)] Guid id, int flags, IntPtr token);
```

调用方法的代码如下

```csharp
var downloadFolderPath = SHGetKnownFolderPath(downloadFolderGuid, 0, IntPtr.Zero);
```

完全的代码如下

```csharp
using System.Runtime.InteropServices;

var downloadFolderGuid = new Guid("374DE290-123F-4565-9164-39C4925E467B");

var downloadFolderPath = SHGetKnownFolderPath(downloadFolderGuid, 0, IntPtr.Zero);
Console.WriteLine(downloadFolderPath);
[DllImport("shell32.dll", CharSet = CharSet.Unicode, ExactSpelling = true, PreserveSig = false)]
static extern string SHGetKnownFolderPath([MarshalAs(UnmanagedType.LPStruct)] Guid id, int flags, IntPtr token);
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7980a73b687c430486843b81596689f809b7add0/HebeefeacuLurnaheehaja) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7980a73b687c430486843b81596689f809b7add0/HebeefeacuLurnaheehaja) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7980a73b687c430486843b81596689f809b7add0
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 7980a73b687c430486843b81596689f809b7add0
```

获取代码之后，进入 HebeefeacuLurnaheehaja 文件夹，即可获取到源代码




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。