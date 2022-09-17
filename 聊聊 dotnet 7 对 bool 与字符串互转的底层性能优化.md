# 聊聊 dotnet 7 对 bool 与字符串互转的底层性能优化

本文也叫 跟着 Stephen Toub 大佬学性能优化系列，大家都知道在 .NET 7 有众多的性能优化，其中就包括了对布尔和字符串互转的性能优化。在对布尔和字符串的转换的性能优化上，有着非常巧妙的思路，值得写篇博客记录

<!--more-->

<!-- 发布 -->

在 [Performance Improvements in .NET 7 - .NET Blog](https://devblogs.microsoft.com/dotnet/performance_improvements_in_net_7/) 这篇官方开发博客里面提到了 dotnet 7 的各个方面的性能优化，其中就包括了本身的主角，对 bool 与 string 互转的性能优化。此优化的核心实现代码请参阅 [https://github.com/dotnet/runtime/pull/64782](https://github.com/dotnet/runtime/pull/64782)

在将 `"True"` 和 `"False"` 字符串和 bool 布尔互相转换时，假定不使用框架内置的转换方法，那将会如何做呢？ 先从字符串转到布尔开始

开始转换的第一步可以通过字符串的长度进行快速的判断。例如无论是 `"True"` 还是 `"False"` 字符串，字符的长度都一定是大于 3 的，因此第一步可以尝试判断一下，是否大于 3 个字符。如果这一条不满足，那肯定不能作为布尔的转换字符串了。如果刚好等于 4 个字符长度，那也许就是 `"True"` 字符串了

转换字符串为布尔时，将无视字符串的大小写，人类方便理解的一个版本如下

```csharp
        internal static bool IsTrueStringIgnoreCase(ReadOnlySpan<char> value)
        {
            return value.Length == 4 &&
                    (value[0] == 't' || value[0] == 'T') &&
                    (value[1] == 'r' || value[1] == 'R') &&
                    (value[2] == 'u' || value[2] == 'U') &&
                    (value[3] == 'e' || value[3] == 'E');
        }
```

没错，就是每个字符串都判断一下。刚好这就是 dotnet 6 所采用的转换方法。同理，判断 `"False"` 也是类似的代码逻辑

```csharp
        internal static bool IsFalseStringIgnoreCase(ReadOnlySpan<char> value)
        {
            return value.Length == 5 &&
                    (value[0] == 'f' || value[0] == 'F') &&
                    (value[1] == 'a' || value[1] == 'A') &&
                    (value[2] == 'l' || value[2] == 'L') &&
                    (value[3] == 's' || value[3] == 'S') &&
                    (value[4] == 'e' || value[4] == 'E');
        }
```

假定要对以上的代码进行性能优化，可以怎么做呢？似乎我想不到有什么方法

在阅读了 [Stephen Toub](https://github.com/stephentoub) 大佬的优化，我才了解到原来还有如此的新思路，通过编码的方式进行优化。刚好的是在 C# 里面字符串的内存存储编码采用的是 Utf16 编码，采用 UTF16 编码的好处在于任意字符所在的内存的偏移量是可以完全瞬时确定的，可以快速和二进制进行映射从而提供更多优化的方向，例如刚好一个 UInt64 也就是一个 ulong 的长度就等于 4 个字符。而刚好 `"True"` 字符串就是 4 个字符

同样感谢远古的 ASCII 编码的定义的超级大佬，英文的大小写转换只需要取一个 0x20 的或即可让字符转换为小写，无论之前的字符是大写还是小写。于是先对输入的字符串内容，从字符串取其中前 4 个字符，转换为 UInt64 的数据，转换方法如下

```csharp
internal static bool IsTrueStringIgnoreCase(ReadOnlySpan<char> value)
{
    var theUInt64Value = BinaryPrimitives.ReadUInt64LittleEndian(MemoryMarshal.AsBytes(value));
    // 忽略代码
}
```

通过 MemoryMarshal.AsBytes 方法，快速将 value 映射为 byte 数组，这里必须说明的是，通过 MemoryMarshal.AsBytes 方法是做内存映射而不是一个转换的过程，这是非常快速且安全的一个过程。接着调用 BinaryPrimitives.ReadUInt64LittleEndian 方法转换为 UInt64 的数据

根据上文的 ASCII 编码规则，为了方便判断逻辑，将转换的结果全部转换为小写的内容，转换方法是使用 `或` 对每个字符进行 `或` 逻辑，代码如下

```csharp
var caseValue = theUInt64Value | 0x0020002000200020;
```

由于字符使用 UTF16 编码，需要每个使用 `0x0020` 进行 `或` 逻辑，连起来就是 `0020_0020_0020_0020` 的数值

接着将 `true` 字符串转换为 UInt64 数据，转换之后的常量的编码是 0x65007500720074 这个数字。刚好进行一次 UInt64 判断即可，耗时可以被忽略

```csharp
return caseValue == 0x65007500720074
```

干掉这些变量，连成一句代码，最终的代码如下

```csharp
        internal static bool IsTrueStringIgnoreCase(ReadOnlySpan<char> value) =>
            value.Length == 4 &&
            (BinaryPrimitives.ReadUInt64LittleEndian(MemoryMarshal.AsBytes(value)) | 0x0020002000200020) == 0x65007500720074; // "true" as a ulong, each char |'d with 0x0020 for case-insensitivity
```

以上代码的 `=>` 是替换原本的方法体的简写代码，原本的代码里面，方法是使用 `{}` 组织方法体代码。在新 C# 里面，如果方法体的代码只有一句话，那就和辣么大一样，用箭头的方式即可，如此可以让代码更加简短，同时方便淘汰那些落后的开发者

同理对 `"False"` 字符串也进行相同的处理。嗯，对于我这个学渣来说，最怕看到“同理”这两个字，因为我拿出脚趾都算不出来，好像 `"False"` 字符串有五个字符吧，大家算算看

那既然有五个字符，这就意味着不能整个字符串转换为 UInt64 了，毕竟 `5 x 16 > 64` 了，那咋办呢？没关系，先取 `"False"` 字符串前面的 `"Fals"` 字符出来，按照上文的方式进行比较，最后再比较 `e` 这个字符好了

```csharp
       internal static bool IsFalseStringIgnoreCase(ReadOnlySpan<char> value) =>
            value.Length == 5 &&
            (((BinaryPrimitives.ReadUInt64LittleEndian(MemoryMarshal.AsBytes(value)) | 0x0020002000200020) == 0x73006C00610066) & // "fals" as a ulong, each char |'d with 0x0020 for case-insensitivity
            ((value[4] | 0x20) == 'e'));
```

这能提升多少呢？非常多。大佬的基准测试如下，可以看到在 .NET 7 的耗时，接近是 .NET 6 的三分之一，优化特别大

|Method | 	Runtime |	Mean| 	Ratio|
|--|--|--|--|
|ParseTrue 	|.NET 6.0 	|7.347 ns 	|1.00|
|ParseTrue 	|.NET 7.0 	|2.327 ns 	|0.32|

在从字符串转换，可以使用整数对比和转换的方法提升性能，那转换为字符串呢？其实也相同，也可以使用相同的方法。嗯，又是同理。同理，在将布尔转换为字符串时，可以通过写入整数的方式提升性能

例如将 true 写为 `"True"` 字符串，原本的写入采用的是如下的方法

```csharp
public bool TryFormat(Span<char> destination, out int charsWritten)
{
    // 这里的 m_value 就是实际存储的字段的值，表示当前的布尔值
    // 这里传入的 destination 则是一段需要被写入的字符内容，咱这个函数就是需要在
    // 传入的 destination 里将布尔值作为 True 或 False 字符串写入
    // 写入成功返回 true 的值，且记录 charsWritten 说明写入了多少个字符了
    ///写入失败，返回 false 的值
    if (m_value)
    {
    	// 在这里需要写入 "True" 字符串到 destination 里
        if ((uint)destination.Length > 3) // 如果长度都还没达到能写入 "True" 字符串的最小所需空间，那就不干活了 
        {
            destination[0] = 'T';
            destination[1] = 'r';
            destination[2] = 'u';
            destination[3] = 'e';
            charsWritten = 4;
            return true;
        }
    }
}
```

上面的代码也看到起来非常方便理解。在了解了可以使用整数的方式提升性能之后，试试换成使用整数赋值的方式，更改之后的代码如下





```csharp
public bool TryFormat(Span<char> destination, out int charsWritten)
{
    // 这里的 m_value 就是实际存储的字段的值，表示当前的布尔值
    // 这里传入的 destination 则是一段需要被写入的字符内容，咱这个函数就是需要在
    // 传入的 destination 里将布尔值作为 True 或 False 字符串写入
    // 写入成功返回 true 的值，且记录 charsWritten 说明写入了多少个字符了
    ///写入失败，返回 false 的值
    if (m_value)
    {
    	// 在这里需要写入 "True" 字符串到 destination 里
        if ((uint)destination.Length > 3) // 如果长度都还没达到能写入 "True" 字符串的最小所需空间，那就不干活了 
        {
            // destination[0] = 'T';
            // destination[1] = 'r';
            // destination[2] = 'u';
            // destination[3] = 'e';
            BinaryPrimitives.WriteUInt64LittleEndian(MemoryMarshal.AsBytes(destination), 0x65007500720054); // "True"
            charsWritten = 4;
            return true;
        }
    }
}
```

先使用 MemoryMarshal.AsBytes 方法将要写入的 destination 当成 byte 二进制，接着使用 BinaryPrimitives.WriteUInt64LittleEndian 将 `"True"` 字符串对应的整数写入到二进制里面去，如此即可进行快速的完成写入字符串

同理，也对 `"False"` 字符串进行写入

```csharp
                    BinaryPrimitives.WriteUInt64LittleEndian(MemoryMarshal.AsBytes(destination), 0x73006C00610046); // "Fals"
                    destination[4] = 'e';
                    charsWritten = 5;
```

如此写入的性能提升也是很多的，大佬的基准测试里面，对比 dotnet 6 的版本，耗时比例是之前的 0.66 倍

|Method | 	Runtime |	Mean| 	Ratio|
|--|--|--|--|
|FormatTrue 	|.NET 6.0 	|3.030 ns 	|1.00|
|FormatTrue 	|.NET 7.0 	|1.997 ns 	|0.66|

在底层里面，对布尔这样的类型进行优化是非常有必要的，可以极大的提升整个上层应用的性能。因为如此底层的逻辑，在很多角落里，无论是否被程序员所关注到，都会被默默的很多次的被调用

既然在性能优化的角度上，通过使用整数替换逐个字符的方式，可以提升一些性能。既然学到了，那肯定要用上来呀，我在文件下载库的断点续传保存文件里面，就尝试使用此思想编写一点代码，代码请看 [https://github.com/dotnet-campus/dotnetCampus.FileDownloader/blob/aa99be3a6c9efe5bc590111a5a4b95085061d622/src/dotnetCampus.FileDownloader/Utils/BreakpointResumptionTransmissions/BreakpointResumptionTransmissionRecordFileFormatter.cs#L128-L136](https://github.com/dotnet-campus/dotnetCampus.FileDownloader/blob/aa99be3a6c9efe5bc590111a5a4b95085061d622/src/dotnetCampus.FileDownloader/Utils/BreakpointResumptionTransmissions/BreakpointResumptionTransmissionRecordFileFormatter.cs#L128-L136)

原本我是期望写入的是 `DCFBPRTI` 的 ASCII 编码的字符串的，现在更换为使用一个 long 代替，如下面代码

```csharp
    private static long GetHeader()
    {
        // 文件头是 dotnet campus File Downloader BreakPointResumptionTransmissionInfo 几个单词的首个字符 DCFBPRTI 缩写的 ASCII 值
        // 刚好将这个 ASCII 的 byte 数组转换为一个 long 的值
        //var headerByteList = System.Text.Encoding.ASCII.GetBytes("DCFBPRTI");
        // var headerByteList = new byte[] { 68, 67, 70, 66, 80, 82, 84, 73 };
        //return BitConverter.ToInt64(headerByteList)
        return 5283938767475196740;
    }
```

以上的代码只是因为我学到了这个方式进行优化，强行想试试而已，不代表着在业务代码里面一定要使用此方式哦

其实在编写代码的时候，以可读性为第一，除非遇到的模块是属于性能敏感的。但愿阅读本文不会带坏一些新手开发者，让新手开发者想着在任何的地方强行使用写整数代替可读性比较高的字符串处理方法