
# dotnet 数组池 ArrayPool 行为记录

本文记录 dotnet 的 ArrayPool 数组池的行为测试

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

## 归还非租用的数组

自己创建一个 buffer 对象，将其归还给到数组池。此时 buffer 不是从数组池借用，但能够归还成功。且在后续租用的时候，可以将其取出来，证明了数组池里面没有要求归还的是借出的，即可以不需要存储借出的数组

代码如下，以下代码是在 dotnet 9 里测试的

```csharp
using System.Buffers;

var buffer = new byte[1024];
ArrayPool<byte>.Shared.Return(buffer);

for (int i = 0; i < 100; i++)
{
    var t = ArrayPool<byte>.Shared.Rent(1024);
    if (ReferenceEquals(t, buffer))
    {
        Console.WriteLine($"归还的自己申请的非租用的 Buffer 数组，可以重新被借用出来");
    }
}
```

预期能够进入 `if (ReferenceEquals(t, buffer))` 分支

本文以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/dd0dc415f50fdfa8d8b7658f8e0cf45d587f2f7d/Workbench/CinarkarnearNeachakahi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/dd0dc415f50fdfa8d8b7658f8e0cf45d587f2f7d/Workbench/CinarkarnearNeachakahi) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin dd0dc415f50fdfa8d8b7658f8e0cf45d587f2f7d
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin dd0dc415f50fdfa8d8b7658f8e0cf45d587f2f7d
```

获取代码之后，进入 Workbench/CinarkarnearNeachakahi 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。