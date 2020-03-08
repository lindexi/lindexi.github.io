# dotnet 使用 lz4net 压缩 Stream 或文件

在 dotnet 可以使用 LZ4 这个无损的压缩算法，这个压缩算法的压缩率不高但是速度很快。这个库支持在 .NET Standard 1.6 .NET Core .NET Framework Mono Xamarin 和 UWP 运行

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


压缩算法 LZ4 的原代码是使用 C 写的，请看[代码](https://github.com/lz4/lz4) 本文的 [K4os.Compression.LZ4](https://github.com/MiloszKrajewski/K4os.Compression.LZ4 ) 是使用 C# 写的，里面也有版本使用了 C++ 代码

因为很多时候都是对 Stream 压缩，所以重点告诉大家如何进行 Stream 压缩

注意这个压缩算法不是 zip 或 rar 压缩，也就是压缩文件不能使用现在的 zip 压缩软件打开，同时压缩的内容也不是文件

使用 NuGet 安装 [K4os.Compression.LZ4.Streams](https://www.nuget.org/packages/K4os.Compression.LZ4.Streams) 很简单就可以使用 LZ4 压缩

如我需要压缩一个字符串到文件

```csharp
using K4os.Compression.LZ4.Streams;

            using (var stream = LZ4Stream.Encode(File.Create("1.lz4")))
            {
                using (var sw = new StreamWriter(stream))
                {
                    sw.WriteLine("林德熙是逗比");
                }
            }

```

这样就将字符串压缩进了文件

调用 LZ4Stream.Encode 传入 stream 对返回的 stream 写入将会压缩到传入的 stream 如上面代码

在解压缩是 LZ4Stream.Decode 方法，如解压缩上面的文件

```csharp
            using (var stream = new StreamReader(LZ4Stream.Decode(File.Open("1.lz4", FileMode.Open))))
            {
                Console.WriteLine(stream.ReadLine());
            }
```

运行代码可以发现输出逗比这就是 LZ4 简单的使用，其实复杂的使用和简单的也差不多

在 Encode 和 Decode 里面还可以传入参数，用于配置更高性能的压缩

[lz4/lz4: Extremely Fast Compression algorithm](https://github.com/lz4/lz4 )

[K4os.Compression.LZ4](https://github.com/MiloszKrajewski/K4os.Compression.LZ4 )

所有代码都在 [github](https://github.com/lindexi/lindexi_gd/tree/c315a9e325e07abe3782a5966d2b24ebd2e92954/DurbujukerhaHaykairyearnal )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
