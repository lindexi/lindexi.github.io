
# dotnet 简单解压缩 tar gz 到文件夹

本文将和大家介绍如何在 dotnet 7 或更高版本里，使用不到 10 行可执行代码解压缩 tar.gz 压缩包到文件夹

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

从 dot.net 官网下载的 linux 系列的 dotnet 运行时包都是 tar.gz 格式的，如 dotnet-runtime-8.0.17-linux-x64.tar.gz 文件。这样的文件在 Windows 下是比较不友好的，不能一次性解压缩。好在 dotnet 7 添加了 `System.Formats.Tar` 命名空间，支持了 tar 格式

假定拿到了 `E:\download\dotnet-runtime-8.0.17-linux-x64.tar.gz` 文件，准备将其解压缩到 CahohuneaFaidinalnalfaji 文件夹，则解压缩的代码如下

```csharp
using System.Formats.Tar;
using System.IO.Compression;

var file = @"E:\download\dotnet-runtime-8.0.17-linux-x64.tar.gz";

var output = Path.Join(AppContext.BaseDirectory, "CahohuneaFaidinalnalfaji");
Directory.CreateDirectory(output);
using var fileStream = File.OpenRead(file);

using var gZipStream = new GZipStream(fileStream, CompressionMode.Decompress);
TarFile.ExtractToDirectory(gZipStream, output, overwriteFiles: true);
```

以上代码的核心是先采用 GZipStream 对输入的 tar.gz 文件进行解压缩，再采用 TarFile 将解压缩之后的内容写入到传入的文件夹里面

如上面代码所示，不到 10 行可执行代码，就可以完成从 tar.gz 压缩文件里读取数据，解压缩到输出文件夹

有一些伙伴也许会错误地使用 TarFile 辅助类直接解压缩 tar.gz 文件，如以下错误示例所示

```csharp
var file = @"E:\download\dotnet-runtime-8.0.17-linux-x64.tar.gz";

var output = Path.Join(AppContext.BaseDirectory, "CahohuneaFaidinalnalfaji");
// System.IO.InvalidDataException:“Unable to parse number.”
TarFile.ExtractToDirectory(file, output, overwriteFiles: true);
```

以上代码将会遇到 System.IO.InvalidDataException:“Unable to parse number.” 错误，因为 tar.gz 文件不是 tar 文件哦。这里是包含了两层打包过程，第一层是 tar 层，将疏松文件夹组织成一个 tar 格式文件。第二层是将 tar 格式文件采用 gz 压缩。这就是为什么直接使用 TarFile 解压缩 tar.gz 会错误的原因，因为 tar.gz 压根就不是 tar 格式，而是 gz 格式的。必须采用 GZipStream 解压缩之后才能给 TarFile 使用

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/cd6a31199f243a9f5410688492931c9d93d6ab40/Workbench/RefaikiwairlurWhanaikeeleda) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/cd6a31199f243a9f5410688492931c9d93d6ab40/Workbench/RefaikiwairlurWhanaikeeleda) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin cd6a31199f243a9f5410688492931c9d93d6ab40
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin cd6a31199f243a9f5410688492931c9d93d6ab40
```

获取代码之后，进入 Workbench/RefaikiwairlurWhanaikeeleda 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。