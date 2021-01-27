
# dotnet C# 将 Byte 二进制数组使用不安全代码快速转换为 int 或结构体数组

我在写一个有趣的 WPF 应用，我会不断收到从硬件发过来的数据，这些数据被使用 Byte[] 数组进行传输。我想要使用最快的方法转换为我的 int 数组或者转换为结构体数组，此时可以使用不安全代码的方式转换

<!--more-->


<!-- 发布 -->

假定有一个二进制数组 Byte[] 是使用如下代码创建的

```csharp
            var memoryStream = new MemoryStream();
            var binaryWriter = new BinaryWriter(memoryStream);
            for (int i = 0; i < 100; i++)
            {
                binaryWriter.Write(i);
            }

            memoryStream.Position = 0;

            var byteList = memoryStream.ToArray();
```

也就是说本质这是一个 int 数组，在获取到 byteList 时，可以如何快速转换为 int 数组使用？如果使用不安全代码，那么转换逻辑将会非常简单

```csharp
            unsafe
            {
                var length = byteList.Length / sizeof(int);
                fixed (byte* bytePointer = byteList)
                {
                    int* intList = (int*) bytePointer;
                    // 这里就获取到了 int 数组，虽然这是一个指针的数组
                    for (int i = 0; i < length; i++)
                    {
                        int value = *intList;
                        Console.WriteLine(value);
                        intList++;
                    }
                }
            }
```

在使用不安全代码的时候，需要在项目属性生成里面勾选允许不安全代码，或者在csproj中添加下面代码

```xml
  <PropertyGroup>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
  </PropertyGroup>
```

而如果此时的 byte 数组的内容实际是某个结构体呢？例如我有一个结构体定义如下

```csharp
        [StructLayout(LayoutKind.Sequential)]
        struct FooStruct
        {
            public int N1 { get; set; }
            public int N2 { get; set; }
            public int N3 { get; set; }
        }
```

使用这个代码写入到二进制

```csharp
            for (int i = 0; i < 100; i++)
            {
                var fooStruct = new FooStruct()
                {
                    N1 = i,
                    N2 = i,
                    N3 = i
                };
                binaryWriter.Write(fooStruct.N1);
                binaryWriter.Write(fooStruct.N2);
                binaryWriter.Write(fooStruct.N3);
            }

            memoryStream.Position = 0;

            byteList = memoryStream.ToArray();
```

此时和上面代码差不多，只是使用对应的结构体强行转换指针就可以

```csharp
            unsafe
            {
                var length = byteList.Length / sizeof(FooStruct);
                fixed (byte* bytePointer = byteList)
                {
                    var fooStructList = (FooStruct*) bytePointer;
                    for (int i = 0; i < length; i++)
                    {
                        var fooStruct = *fooStructList;

                        // 此时就获取到了结构体数组
                    }
                }
            }
```

通过这个方法，虽然是不安全的代码，但是能提升很多性能

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/f897f444/LawdalenaLifearjanugear ) 欢迎小伙伴访问






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。