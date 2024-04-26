本文将告诉大家一个在 ZipArchive 里追加文件夹，以及添加过滤文件处理的压缩文件辅助方法

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

实现的方法的代码如下

```csharp
    /// <summary>
    /// 追加文件夹到压缩文件里面
    /// </summary>
    /// <param name="archive"></param>
    /// <param name="sourceDirectoryName"></param>
    /// <param name="zipRelativePath">在压缩包里面的相对路径</param>
    /// <param name="compressionLevel"></param>
    /// <param name="fileCanAddedPredicate"></param>
    public static void AppendDirectoryToZipArchive(ZipArchive archive, string sourceDirectoryName, string zipRelativePath, CompressionLevel compressionLevel = CompressionLevel.Fastest, Predicate<string>? fileCanAddedPredicate = null)
    {
        var folders = new Stack<string>();

        folders.Push(sourceDirectoryName);

        while (folders.Count > 0)
        {
            var currentFolder = folders.Pop();

            foreach (var item in Directory.EnumerateFiles(currentFolder))
            {
                if (fileCanAddedPredicate != null && !fileCanAddedPredicate(item))
                {
                    continue;
                }

                archive.CreateEntryFromFile(item, Path.Join(zipRelativePath, Path.GetRelativePath(sourceDirectoryName, item)), compressionLevel);
            }

            foreach (var item in Directory.EnumerateDirectories(currentFolder))
            {
                folders.Push(item);
            }
        }
    }
```

演示的调用的代码如下

```csharp
var zipFile = "1.zip";
using (var fileStream = new FileStream(zipFile, FileMode.Create, FileAccess.Write))
{
    using var zipArchive = new ZipArchive(fileStream, ZipArchiveMode.Create, leaveOpen: true/*自己释放 FileStream 对象*/, Encoding.UTF8);
    Foo.AppendDirectoryToZipArchive(zipArchive, @"C:\lindexi\Library\", "Lib");
    Foo.AppendDirectoryToZipArchive(zipArchive, @"C:\lindexi\CA\", "Pem", fileCanAddedPredicate: filePath =>
    {
        var fileName = Path.GetFileName(filePath);
        return fileName != "foo.ignore.file";
    });
}
```

支持设置文件夹加入之后在安装包的什么相对路径下，也支持过滤文件

如果加入到安装包的根路径下，只需要让 zipRelativePath 参数传入空字符串即可，如下面代码

```csharp
    Foo.AppendDirectoryToZipArchive(zipArchive, @"C:\lindexi\Library\", "");
```

全部的代码如下

```csharp
using System.IO.Compression;
using System.Text;

var zipFile = "1.zip";
using (var fileStream = new FileStream(zipFile, FileMode.Create, FileAccess.Write))
{
    using var zipArchive = new ZipArchive(fileStream, ZipArchiveMode.Create, leaveOpen: true/*自己释放 FileStream 对象*/, Encoding.UTF8);
    Foo.AppendDirectoryToZipArchive(zipArchive, @"C:\lindexi\Library\", "Lib");
    Foo.AppendDirectoryToZipArchive(zipArchive, @"C:\lindexi\CA\", "Pem", fileCanAddedPredicate: filePath =>
    {
        var fileName = Path.GetFileName(filePath);
        return fileName != "foo.ignore.file";
    });
}


class Foo
{
    /// <summary>
    /// 追加文件夹到压缩文件里面
    /// </summary>
    /// <param name="archive"></param>
    /// <param name="sourceDirectoryName"></param>
    /// <param name="zipRelativePath">在压缩包里面的相对路径</param>
    /// <param name="compressionLevel"></param>
    /// <param name="fileCanAddedPredicate"></param>
    public static void AppendDirectoryToZipArchive(ZipArchive archive, string sourceDirectoryName, string zipRelativePath, CompressionLevel compressionLevel = CompressionLevel.Fastest, Predicate<string>? fileCanAddedPredicate = null)
    {
        var folders = new Stack<string>();

        folders.Push(sourceDirectoryName);

        while (folders.Count > 0)
        {
            var currentFolder = folders.Pop();

            foreach (var item in Directory.EnumerateFiles(currentFolder))
            {
                if (fileCanAddedPredicate != null && !fileCanAddedPredicate(item))
                {
                    continue;
                }

                archive.CreateEntryFromFile(item, Path.Join(zipRelativePath, Path.GetRelativePath(sourceDirectoryName, item)), compressionLevel);
            }

            foreach (var item in Directory.EnumerateDirectories(currentFolder))
            {
                folders.Push(item);
            }
        }
    }
}
```

以上的 `C:\lindexi\Library` 等文件夹是我的用于测试的文件夹，还请大家换成自己的文件夹

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/76bed002b4da4f363037c2d39f41596be1c2b177/LebenehainaiJelearlowiwaw) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/76bed002b4da4f363037c2d39f41596be1c2b177/LebenehainaiJelearlowiwaw) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 76bed002b4da4f363037c2d39f41596be1c2b177
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 76bed002b4da4f363037c2d39f41596be1c2b177
```

获取代码之后，进入 LebenehainaiJelearlowiwaw 文件夹，即可获取到源代码
