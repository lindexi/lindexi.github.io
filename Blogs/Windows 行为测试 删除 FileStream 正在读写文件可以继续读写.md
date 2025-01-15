---
title: Windows 行为测试 删除 FileStream 正在读写文件可以继续读写
description: 本文在 Win11 系统下，测试使用 FileStream 对文件进行读写，读写过程中，删除正在读写的文件后的行为
tags: 
category: 
---

<!-- 发布 -->
<!-- 博客 -->

测试结论：

使用 FileShare 带 Delete 的共享方式打开的 FileStream 正在对文件进行读写过程中，可以对正在读写的文件进行删除。文件删除之后，不影响已经打开的 FileStream 的读写。且此时使用 File.Exists 方法判断文件是否存在，返回的是不存在

如果删除之后，再次在外部新建同名文件，则原本已经打开的 FileStream 对此同名文件毫无影响

如果对文件进行移动（相同驱动器）或重命名，则原本已经打开的 FileStream 能够作用在移动之后的文件下，即写入时能写入到被移动的文件里而不会在原本的路径下重新新建文件再写入内容

以下是具体测试逻辑：

```csharp
var filePath = @"F:\temp\FelernihearBechanalwhi";

await using var fileStream = new FileStream(filePath, FileMode.Create, FileAccess.ReadWrite,
    FileShare.Delete | FileShare.ReadWrite, bufferSize: 10, FileOptions.WriteThrough);

await using var readStream = new FileStream(filePath, FileMode.Open, FileAccess.Read, FileShare.Delete | FileShare.ReadWrite, bufferSize: 10, FileOptions.WriteThrough);

for (var i = 0; i < int.MaxValue; i++)
{
    fileStream.Write([1, 2, 3]);
    await fileStream.FlushAsync();
    Console.ReadLine();

    if (i == 3)
    {
        File.Delete(filePath);
    }

    readStream.Position = 0;
    var buffer = new byte[1024];
    var readCount = readStream.Read(buffer, 0, buffer.Length);
    Console.WriteLine($"文件存在： {File.Exists(filePath)} 读取内容： {readCount}");
}

Console.WriteLine("Hello, World!");
```

以上代码先使用 FileShare.Delete 方式打开了 FileStream 进行读写

再进行三次读写之后，删除文件。经过实际运行测试，可以看到控制台输出内容如下

```
文件存在： True 读取内容： 3

文件存在： True 读取内容： 6

文件存在： True 读取内容： 9

文件存在： False 读取内容： 12

文件存在： False 读取内容： 15
```

通过以上的控制台输出可以看到，使用 File.Exists 确实判断文件不存在了

证明文件确实被删除了，且文件被删除了不影响已打开的 FileStream 的读写

如果大家想要测试移动文件之后的行为，可以自行在资源管理器里面，将文件在同驱动器进行移动或重命名

为什么不能跨驱动器？因为跨驱动器的移动实际上是复制新的和删除原来的，这就不符合实验要求了

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0f361e0d7d984508e8ca19e9414b01074e220516/Workbench/HibairjukegoYekallgera) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0f361e0d7d984508e8ca19e9414b01074e220516/Workbench/HibairjukegoYekallgera) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0f361e0d7d984508e8ca19e9414b01074e220516
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0f361e0d7d984508e8ca19e9414b01074e220516
```

获取代码之后，进入 Workbench/HibairjukegoYekallgera 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
