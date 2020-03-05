# dotnet 使用 MessagePack 序列化对象

和很多序列化库一样，可以通过 MessagePack 序列化和反序列化，和 json 相比这个库提供了二进制的序列化，序列化之后的内容长度比 json 小很多

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


这个库能序列的内容不多，大多数时候建议使用的序列的类都是只有基础的 int 和 bool 字符串等，最好不要存在复杂的类

使用 MessagePack 的好处只是序列化出来的内容的长度小，但是从性能等方面，其实和 Json 差别不大，在序列化简单的类的时候，可以看到 MessagePack 的序列化速度会比较快。在序列化比较大的类如果序列化到文件，那么因为文件读写的性能，可以看到 MessagePack 的性能明显比 json 好。

在使用 MessagePack 之前需要通过 Nuget 安装

安装方法是在 Nuget 输入 MessagePack 安装

下面写一个简单的方法，将会对这个类序列化

```csharp
    [MessagePackObject]
    public class Foo
    {
        [Key(0)]
        public string Lindexi { set; get; }
    }
```

现在主函数创建这个类

```csharp
            var foo = new Foo { Lindexi = "林德熙是逗比" };

```

通过 MessagePackSerializer.Serialize 方法可以将一个类序列化为 byte 数组，或序列化到 stream 也就是可以直接序列化到文件

```csharp
            var byteList = MessagePackSerializer.Serialize(foo);

```

如果想要看 byteList 的内容，可以使用下面方法将 byte 数组转字符串

```csharp
            Console.WriteLine(ByteListToString(byteList));

        private static string ByteListToString(byte[] byteList)
        {
            return string.Concat(byteList.Select(temp => temp.ToString("x2")));
        }
```

使用下面代码可以反序列化

```csharp
            foo = MessagePackSerializer.Deserialize<Foo>(byteList);

            Console.WriteLine(foo.Lindexi);
```

我尝试运行代码，可以看到下面代码

```csharp
91b2e69e97e5beb7e78699e698afe98097e6af94
林德熙是逗比
```

前面是将 Foo 序列化的二进制

如果在使用的时候发现下面代码，那么很多时候都是因为没有在类上面添加特性，需要修改类为公开的，然后在类上面添加 `MessagePackObject` 特性，然后在每个公开属性上面添加 `Key` 特性，同时输入这个属性是在哪个顺序

```csharp
MessagePack.FormatterNotRegisteredException:“KouhoofamerNeejirstistedrea.Foo is not registered in this resolver. resolver:StandardResolver”
```	

github：[msgpack/msgpack-cli](https://github.com/msgpack/msgpack-cli)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
