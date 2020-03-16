# C# dotnet 将 Stream 保存到文件的方法

在拿到一个 Stream 如何优雅将这个 Stream 保存到代码

<!--more-->
<!-- CreateTime:2020/3/14 9:22:48 -->

<!-- 发布 -->

最优雅的方法应该是通过 CopyTo 或 CopyToAsync 的方法

```csharp
using (var fileStream = File.Create("C:\\lindexi\\File.txt"))
{
    inputStream.Seek(0, SeekOrigin.Begin);
    iputStream.CopyTo(fileStream);
}
```

这里的 `inputStream.Seek(0, SeekOrigin.Begin);` 不一定需要，请根据你自己的需求，如你只需要将这个 Stream 的从第10个byte开始复制等就不能采用这句代码

用异步方法会让本次写入的时间长一点，但是会让总体性能更好，让 CPU 能处理其他任务

```csharp
using (var fileStream = File.Create("C:\\lindexi\\File.txt"))
{
    await iputStream.CopyToAsync(fileStream);
}
```

注意使用 CopyToAsync 记得加上 await 哦，执行到这句代码的时候，就将执行交给了 IO 了，大部分的 IO 处理都不需要 CPU 进行计算，这样能达到总体性能更好

另外如果 iputStream 是外面传入的，那么我不建议在这个方法里面释放，为什么呢？我用的好好的一个Stream传入一个业务就被干掉了

其次的方法是自己控制内存复制缓存，此方法将会多出一次内存复制

```csharp
public static void CopyStream(Stream input, Stream output)
{
    byte[] buffer = new byte[1024];
    int len;
    while ( (len = input.Read(buffer, 0, buffer.Length)) > 0)
    {
        output.Write(buffer, 0, len);
    }    
}

// 使用方法如下
using (Stream file = File.Create("C:\\lindexi\\File.txt"))
{
    CopyStream(input, file);
}
```
 
此方法的作用就是让你修改 `new byte[1024]` 的值，让你可以控制复制的缓存

接下来就是一些不推荐的方法了，但是写的时候方便

```csharp
using (var stream = new MemoryStream())
{
    input.CopyTo(stream);
    File.WriteAllBytes(file, stream.ToArray());
}
```

上面这个方法将会复制两次内存，而且如果 input 这个资源长度有 1G 就要占用 2G 的资源

和上面差不多的是申请一个大的缓存，如下面代码

```csharp
public void SaveStreamToFile(string fileFullPath, Stream stream)
{
    if (stream.Length == 0) return;

    using (FileStream fileStream = System.IO.File.Create(fileFullPath, (int)stream.Length))
    {
        byte[] bytesInStream = new byte[stream.Length];
        stream.Read(bytesInStream, 0, (int)bytesInStream.Length);

        fileStream.Write(bytesInStream, 0, bytesInStream.Length);
     }
}
```

从效率和代码的优雅其实都不如 CopyTo 方法，而且因为 stream.Length 作为长度没有决定缓存，所以也不如第二个方法

下面是一个超级慢的方法，一个 byte 一个 byte 写入的速度是超级慢的

```csharp
public void SaveStreamToFile(Stream stream, string filename)
{  
   using(Stream destination = File.Create(filename))
   {
       Write(stream, destination);
   }
}


public void Write(Stream from, Stream to)
{
      for(int a = from.ReadByte(); a != -1; a = from.ReadByte())
      {
      	to.WriteByte( (byte) a );
      }
}
```

[.net - How do I save a stream to a file in C#? - Stack Overflow](https://stackoverflow.com/questions/411592/how-do-i-save-a-stream-to-a-file-in-c )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
