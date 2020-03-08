# dotnet 使用 GZipStream 压缩字符串

在 dotnet 提供了 GZipStream 进行方便的 Stream 压缩，可以用这个方法压缩字符串

<!--more-->
<!-- CreateTime:2020/1/17 15:51:18 -->

<!-- 发布 -->

先拿到字符串，然后使用 Encoding.UTF8 转换为 byte 数组，写入到压缩

```csharp
            var byteList = Encoding.UTF8.GetBytes(str);
```

上面的 str 就是传入的需要压缩的字符串

使用 GZipStream 需要先传入存放压缩的 Stream 可以创建内存 Stream 等

```csharp
            var memoryStream = new MemoryStream();
            var gZipStream = new GZipStream(memoryStream, CompressionMode.Compress);
```

通过修改 CompressionMode.Compress 就可以设置为压缩或解压缩

对 gZipStream 写入内容，写入完成之后关闭，就会自动将写入的内容压缩

```csharp
            var gZipStream = new GZipStream(memoryStream, CompressionMode.Compress);
            var byteList = Encoding.UTF8.GetBytes(str);
            gZipStream.Write(byteList, 0, byteList.Length);
            gZipStream.Close();
```

获取压缩后的数据可以通过 `memoryStream.ToArray` 方法

全部代码

```csharp
            var str = "123123123123123123123123123123123123";
            var memoryStream = new MemoryStream();
            var gZipStream = new GZipStream(memoryStream, CompressionMode.Compress);
            var byteList = Encoding.UTF8.GetBytes(str);
            gZipStream.Write(byteList, 0, byteList.Length);
            gZipStream.Close();
            var output = memoryStream.ToArray();
```

解压缩可以在获取到 byte[] 转换为 MemoryStream 然后通过 GZipStream 读取

```csharp
            var memoryStream = new MemoryStream(xx);
            var gZipStream = new GZipStream(memoryStream, CompressionMode.Decompress);
```

可以通过 BinaryReader 辅助读取

```csharp
            var binaryReader = new BinaryReader(gZipStream, Encoding.UTF8);
            str = Encoding.UTF8.GetString(binaryReader.ReadBytes(10000));
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
