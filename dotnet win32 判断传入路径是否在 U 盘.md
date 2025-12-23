# dotnet win32 判断传入路径是否在 U 盘

我在制作一个类似 PPT 的工具，这个工具有超链接模块，我需要关注的是超链接文件是否链接到 U 盘上了已给出提示。防止一些用户链接到自己电脑上，然后换个电脑又找不到

<!--more-->
<!-- CreateTime:2025/12/20 07:07:41 -->

<!-- 发布 -->
<!-- 博客 -->

通过 dotnet 自带的 DriveInfo 类就能够很好地实现这一点。只需判断 DriveType 属性是否为 Removable 即可了解是否是 U 盘。判断原理是判断可插拔的盘

遍历当前系统的所有磁盘，判断有哪些是 U 盘的方法如下

```csharp
foreach (var driveInfo in DriveInfo.GetDrives())
{
    if (driveInfo.DriveType == DriveType.Removable)
    {
        Console.WriteLine($"{driveInfo.RootDirectory} 是 U 盘");
    }
}
```

点开 DriveType 还可以看到更多有趣的类型，比如光盘等

判断传入路径是否在 U 盘里，可用如下判断

```csharp
var path = @"H:\lindexi\test.txt";
var isUDiskPath = IsUDiskPath(path);
Console.WriteLine($"Path={path} 是 U 盘={isUDiskPath}");

static bool IsUDiskPath(string path)
{
    if (!Path.IsPathFullyQualified(path))
    {
        throw new ArgumentException($"路径必须是绝对路径。 Path={path}", nameof(path));
    }

    var pathRoot = Path.GetPathRoot(path);
    if (pathRoot is null)
    {
        return false;
    }

    var driveInfo = new DriveInfo(pathRoot);
    // 必须先判断 IsReady 属性，详细请看 [C# 获取磁盘或硬盘信息的坑，存在未就绪（IsReady = false）导致异常的问题 - wuty007 - 博客园](https://www.cnblogs.com/wuty/p/18413323 )
    return driveInfo.IsReady && driveInfo.DriveType == DriveType.Removable;
}
```

软软对 DriveInfo 的容错设计是非常好的，接受的输入非常多：

- 支持传入整个路径：如 `H:\lindexi\test.txt` 也是可以的
- 支持传入标准根路径：如 `H:\`
- 支持传入盘符：如 `H`
- 支持传入不存在的磁盘路径，如 `Y:` 盘。此时不会抛出异常，只会让 DriveType 为 NoRootDirectory 类型

以上代码预先判断了 IsReady 属性，随后再获取 DriveType 属性，以减少出现未就绪异常。详细请看 [C# 获取磁盘或硬盘信息的坑，存在未就绪（IsReady = false）导致异常的问题 - wuty007 - 博客园](https://www.cnblogs.com/wuty/p/18413323 )

本文的核心代码如下

```csharp
var path = @"H:\lindexi\test.txt";
var isUDiskPath = IsUDiskPath(path);
Console.WriteLine($"Path={path} 是 U 盘={isUDiskPath}");

foreach (var driveInfo in DriveInfo.GetDrives())
{
    if (driveInfo.DriveType == DriveType.Removable)
    {
        Console.WriteLine($"{driveInfo.RootDirectory} 是 U 盘");
    }
}
Console.WriteLine("Hello, World!");

static bool IsUDiskPath(string path)
{
    if (!Path.IsPathFullyQualified(path))
    {
        throw new ArgumentException($"路径必须是绝对路径。 Path={path}", nameof(path));
    }

    var pathRoot = Path.GetPathRoot(path);
    if (pathRoot is null)
    {
        return false;
    }

    var driveInfo = new DriveInfo(pathRoot);
    // 必须先判断 IsReady 属性，详细请看 [C# 获取磁盘或硬盘信息的坑，存在未就绪（IsReady = false）导致异常的问题 - wuty007 - 博客园](https://www.cnblogs.com/wuty/p/18413323 )
    return driveInfo.IsReady && driveInfo.DriveType == DriveType.Removable;
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/9724bf15f3abdc22d3e47f17c88e9067734b329d/Workbench/CawrelibairquJojaijurhewe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/9724bf15f3abdc22d3e47f17c88e9067734b329d/Workbench/CawrelibairquJojaijurhewe) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 9724bf15f3abdc22d3e47f17c88e9067734b329d
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 9724bf15f3abdc22d3e47f17c88e9067734b329d
```

获取代码之后，进入 Workbench/CawrelibairquJojaijurhewe 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )