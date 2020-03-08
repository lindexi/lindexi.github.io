# UWP IRandomAccessStream 与 Stream 互转

本文告诉大家如何将 IRandomAccessStream 和 Stream 互转

<!--more-->
<!-- CreateTime:2019/1/29 16:33:30 -->


<div id="toc"></div>

如果在使用网络传输文件的时候，在 UWP 经常使用将 IRandomAccessStream 和 Stream 互转。

因为在 UWP 使用打开文件作为流的最简单代码如下

```csharp
IRandomAccessStream randomAccessStream = await File.OpenAsync(FileAccessMode.Read)
```

这时通过 `using System.IO;` 可以使用扩展方法将 IRandomAccessStream 转 Stream 请看下面代码

```csharp
using System.IO;

var stream = randomAccessStream.AsStream();
```

如果需要反过来将 stream 转 IRandomAccessStream 也是同样需要引用 `using System.IO;` 来让代码可以写出扩展方法

```csharp
using System.IO;

IRandomAccessStream randomAccessStream = stream.AsRandomAccessStream();
```

https://stackoverflow.com/a/33221178/6116637

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
