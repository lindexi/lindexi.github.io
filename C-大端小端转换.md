
# C# 大端小端转换

关于大端和小端，是一个有趣的问题。本文告诉大家如何在C#转换大端和小端。

<!--more-->


<!-- CreateTime:2019/8/31 16:55:58 -->


这里有一个有趣的故事，请看[详解大端模式和小端模式 - CSDN博客](https://blog.csdn.net/ce123_zhouwei/article/details/6971544 )

默认的 C# 使用的是小端，如果收到的消息是大端，那么就会出现解析错误。

例如收到的数据是 byte 数组，现在知道数据是大端数据，需要把大端转小端，有以下的几个方法进行转换。本文将按照推荐的优先级的顺序告诉大家这几个不同的方法

## 使用 BinaryPrimitives 辅助转换

这个 BinaryPrimitives 类型是从 .NET Standard 2.1 引入的，对于版本号大于等于 .NET Core 2.1 的，包括 .NET 5 和 .NET 6 和 .NET 7 等框架都是可以使用的。通过 BinaryPrimitives 静态类型提供的辅助方法，可以进行明确的大端和小端读取转换。传入参数支持采用 `ReadOnlySpan<byte>` 类型，也就是截取数组的某一段空间，可以实现不额外分配内存（堆空间内存）进行转换

例如从传入的 Stream 里面读取一段内存，已知这是一段采用大端存放的 UInt32 的数据，那么读取的例子如下。大家可以将以下的关于 UInt32 也就是 uint 类型更换为你自己期望读取的类型

先从数组池租借一段空间。如果这个业务只是很少用到的，以及用到时，只会使用一次，也不是性能敏感的地方，那直接创建数组也不错，毕竟读取一个 UInt32 需要的数组也足够小，读取完成之后将会被 GC 通过引用计数释放空间

```csharp
var buffer = ArrayPool<byte>.Shared.Rent(sizeof(uint));
```

获取到读取的数据的占用的空间大小

```csharp
int count = sizeof(uint);
```

这里不推荐大家背某个基础类型占用多少内存空间，十分推荐应该采用 sizeof 方法获取。原因有两个，第一个就是如果自信过头，背错了呢，相信我，你现在没背错，等你加班到我现在这个时间点，嘿嘿。第二个原因就是做平台迁移也许有坑，尽管经过了几十年都不会有此问题，但谁知道后面是否也许会遇到变更呢，当年的程序员也说 Ascii 就足够用了

将缓存 buffer 转换为 `Span<byte>` 进行读取，如果缓存 buffer 是从数组池租借的，那这一步非常有必要。原因是数组池给的数组一定是大于等于说需要的长度的，换句话说，你告诉数组池要一段长度为 10 的数组，数组池说，现在长度为 10 的借完了，但是存货还有一个长度为 1000 的，那就先给你这个吧，于是拿到的数组长度就远大于所需要的长度。转换为 `Span<byte>` 可以让其他业务忽略数组长度细节，减少逻辑复杂度

```csharp
var bufferSpan = buffer.AsSpan(0, count);
```

接着从 Stream 里面读取出内容

```csharp
var readCount = Stream.Read(bufferSpan);
```

对于一个 Stream 来说，判断读取到的长度是特别有必要的。例如读取一个文件，期望是读取到 10 的长度，实际上这个文件的长度只有 5 那么证明此读取是失败的。根据具体业务需求，决定是通过返回值告诉上层，还是抛出异常。大部分这个情况下，抛出 EndOfStreamException 也算合理

```csharp
        if (readCount != count)
        {
            throw new EndOfStreamException();
        }
```

如果选择抛出异常了，记得让上层处理

读取到了数据，接下来就可以调用 BinaryPrimitives 静态类提供的方法进行转换

```csharp
uint value = BinaryPrimitives.ReadUInt32BigEndian(bufferSpan);
```

有借有还，如果是从数组池租借的，记得还给数组池

```csharp
ArrayPool<byte>.Shared.Return(buffer);
```

通过此方法即可完成大端的转换。此方法的优点在于性能较高，不需要有多余的数组申请和反转。对于转换其他类型，例如 int 这些，方法也差不多，只需要将上面的 uint 和 UInt32 替换为 int 等即可

## 数组反转法

这是比较传统的方法，可以用在 .NET 比较古老的版本上。缺点是需要拷贝数组以及做数组的反转，相对来说，性能没有那么好。但，敲黑板，就反转几个 byte 而已，这里说性能差，是相对的，实际上差不多哪儿去

数组反转法的第一步是需要把数据复制出来

### 复制数组

假设收到的数据是 data ，里面的前两个 byte 是不需要的，格式如下图

![](http://image.acmx.xyz/lindexi%2F2018528102650406.jpg)

也就是需要复制出第2个到第5个byte出来，转换这个数据反序。

复制数组的方式有很多个，例如 Array.Copy 和 Buffer.BlockCopy 这两个函数，两个函数使用方式差不多

下面我使用 Array.Copy 做例子

首先定义一个数组用来反序，如果是频繁使用的话，推荐将这个数组缓存起来

```csharp
var revertByteList = new byte[4];
```

然后复制数据

```csharp
Array.Copy(data, 2, revertByteList, 0, 4);
```

对数据反序，这样就转换大端

```csharp
revertByteList = revertByteList.Reverse().ToArray();
```

### 数组转整数

从数组转整数的方式很简单，使用下面代码就可以转换

```csharp
var n = BitConverter.ToInt32(revertByteList, 0);
```

小端转大端就是先把大端存储的 int 转 byte 数组，然后按照每 4 个 byte 反序就可以





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。