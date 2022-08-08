
# dotnet 解析 TTF 字体文件格式

在 Windows 下，可以使用 DX 提供的强大能力，调用 DX 读取 TTF 字体文件，获取字体文件的信息以及额外的渲染信息。特别是基于 DX 的 WPF 更是加了一层封装，使用 FontFamily 类型提供的友好方法获取到字体的信息。出于学习的目的，本文将不使用任何平台封装好的方法，自己读取二进制的 TTF 文件，解析 TTF 的内容，获取到字体文件里面的字体名

<!--more-->



<!-- 发布 -->
<!-- 博客 -->

在 Windows 下，使用 WPF 获取字体信息的方法请看 [WPF 从文件加载字体](https://blog.lindexi.com/post/WPF-%E4%BB%8E%E6%96%87%E4%BB%B6%E5%8A%A0%E8%BD%BD%E5%AD%97%E4%BD%93.html )

本文接下来将采用自己读取二进制的 TTF 文件的方法，来告诉大家 TTF 文件的格式

在 TTF 标准里面，前面的 4 个 byte 表示的是 TTF 头信息，可以通过这 4 个 byte 判断此文件是否 TTF 文件。当然，文件头的判断方式只能是说符合条件的可能是 TTF 文件，不符合条件的一定不是 TTF 文件

在开始写代码之前，有一点需要了解的是二进制存储的坑，那就是关于鸡蛋从大的一头开始吃还是从小的一头开始吃的大小端问题。在 TTF 文件里面，采用的是大端的存储方式。为了解决此问题，咱先造一点辅助的代码，用于做大端的转换。关于二进制编码里面的大端和小端，请看我博客 [C# 大端小端转换](https://blog.lindexi.com/post/C-%E5%A4%A7%E7%AB%AF%E5%B0%8F%E7%AB%AF%E8%BD%AC%E6%8D%A2.html)

写一个叫 BigEndianBinaryReader 的类型继承 BinaryReader 类型，重写读取数据的方法，从而实现从大端进行读取，核心采用 BinaryPrimitives 提供的读取大端存储的二进制数据的各个辅助方法，如 `BinaryPrimitives.ReadInt16BigEndian` 等。这个辅助类型非本文重点，如有兴趣，还请到文末获取本文所用全部源代码

新建一个叫 TtfInfo 的类型，此类型将用来作为读取的入口。根据水果家的文档，嗯，这是全网看起来写的最好也最全的文档： [Fonts - TrueType Reference Manual - Apple Developer](https://developer.apple.com/fonts/TrueType-Reference-Manual/)

可以了解到 TTF 字体文件，也就是 TrueType 字体文件里面，首先将放置一个 OffsetTable 用来记录字体里面多个维度的信息存放的地方。在开始读取之前，先读取一下字体的文件头信息，也就是 SfntVersion 信息，如水果家的文档的所示： [Font Tables - TrueType Reference Manual - Apple Developer](https://developer.apple.com/fonts/TrueType-Reference-Manual/RM06/Chap6.html)

读取之前，先定义数据结构，用来表示 TTF 文件信息的版本。这里需要补充一点的是，大部分的二进制数据是不具备自描述能力的，这和 XML 和 JSON 等有很大的不同。大部分的二进制数据是需要由约定定义数据的存储格式，而约定本身是不稳定的，也许会有多个不同的版本的约定，这就是所谓文件信息的版本的概念。一般设计上，在数据格式的约定版本变更时，都会变更其文件信息的版本。当然，这也不是说二进制数据是不能具备自描述能力的，只是业界大部分的二进制数据存储都是追求体积和效率，如果加上了自描述能力，无疑会增加二进制体积以及加了一些解析需要处理的数据量，而且既然有自描述的需求，换用 XML 和 JSON 不香么

```csharp
public readonly record struct Version(ushort Major, ushort Minor)
{
    public static Version Read(BinaryReader reader)
    {
        var major = reader.ReadUInt16();
        var minor = reader.ReadUInt16();
        return new Version(major, minor);
    }

    public override string ToString() => $"{Major}.{Minor}";
}
```

以上传入的 BinaryReader 是本文上面定义的 BigEndianBinaryReader 类型，为了解决 TTF 采用大端存储。以上代码采用了 C# 9 的 record 关键字，详细请看 [使用记录类型 - C# 教程 Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/csharp/whats-new/tutorials/records)

尽管定义上我是分了 Major 和 Minor 两个属性，这在远古时代时，是非常合理的，然而规则就是用来破坏的… 有大佬觉得，既然有 4 个 byte 的空间，那为什么不放个字符串好呢，放个 `1.0` 太浪费了，于是，在 2022 时的判断应该是如下

```csharp
        var sfntVersion = Version.Read(reader);

        // 版本是以下三个之一
        // 0x00010000 对应 1.0 版本
        // $"{(char) 0x74}{(char) 0x74}{(char) 0x63}{(char) 0x66}" = "ttcf" 版本是 ttcf 字符串
        // $"{(char) 0x74}{(char) 0x72}{(char) 0x75}{(char) 0x65}" = "true" 版本是 true 字符串
        if
        (
            (sfntVersion.Major == 0x0001 && sfntVersion.Minor == 0x0000)
            || (sfntVersion.Major == 0x7474 && sfntVersion.Minor == 0x6366)
            || (sfntVersion.Major == 0x7472 && sfntVersion.Minor == 0x7565)
        )
        {
            // 这是合法的 ttf 文件
        }
        else
        {
            // 这是假装是 TTF 的
        }
```

判断文件头是 `0x0001_0000` 表示的是 1.0 版本，很合理。但是也要判断是 `0x7474` 和 `0x6366` 这两个数值，这两个数值其实是 `ttcf` 字符串的各个字符的 Ascii 合起来。另外，还有取 TTF 字体的 TrueType 的 `true` 的各个字符的 Ascii 合起来的 `0x7472` 和 `0x7565` 两个数

尽管文件头不相同，好在里面的内容似乎也没有什么变化

继续读取 TTF 文件信息，除了文件头之外的其他信息就是 OffsetTable 内容

为了方便读取的数据的存放，定义 OffsetTable 类型

```csharp
public readonly record struct OffsetTable(Version SfntVersion, ushort NumTables, ushort SearchRange,
    ushort EntrySelector, ushort RangeShift)
{

}
```

按照顺序读取各个属性，其中将会用到的属性是 NumTables 表示的是这个字体有多少个 Table 需要读取。每个 Table 就是一个维度的信息，例如本文重点读取的 `name` 信息就是存放在其中一个叫 `name` 的 Table 里。每个 Table 都有一个用 4 个 ascii 字符组成的名称，通过水果家的文档可以看到有以下的 Table 信息： [Apple Advanced Typography Font Tables - TrueType Reference Manual - Apple Developer](https://developer.apple.com/fonts/TrueType-Reference-Manual/RM06/Chap6Tables.html)

读取 OffsetTable 的代码如下

```csharp
    public static OffsetTable Read(BinaryReader reader)
    {
        var sfntVersion = Version.Read(reader);

        // 版本是以下三个之一
        // 0x00010000 对应 1.0 版本
        // $"{(char) 0x74}{(char) 0x74}{(char) 0x63}{(char) 0x66}" = "ttcf" 版本是 ttcf 字符串
        // $"{(char) 0x74}{(char) 0x72}{(char) 0x75}{(char) 0x65}" = "true" 版本是 true 字符串
        if
        (
            (sfntVersion.Major == 0x0001 && sfntVersion.Minor == 0x0000)
            || (sfntVersion.Major == 0x7474 && sfntVersion.Minor == 0x6366)
            || (sfntVersion.Major == 0x7472 && sfntVersion.Minor == 0x7565)
        )
        {
            return new OffsetTable(sfntVersion, reader.ReadUInt16(), reader.ReadUInt16(), reader.ReadUInt16(),
                reader.ReadUInt16());
        }
        else
        {
            // 这不是一个 TTF 文件
            throw new ArgumentException("别闹，这不是一个 TTF 文件");
        }
    }
```

通过 NumTables 属性可以获取到这个字体有多少个 Table 信息，由于不同的 Table 存放的数据的长度是不同的，为了方便索引，在 TTF 的存放里面，在 OffsetTable 之后紧存放了各个 Table 的索引信息，索引包含的是各个 Table 的由 4 个 ascii 组成的名称，也就是 Tag 属性，和每个 Table 的校验信息，存放在 TTF 文件的绝对偏移量和长度。多个 Table 的索引之间是连续存放，接下来可以将这些 Table 的索引读取到内存

开始读取先定义 Table 的索引类型为 TableDirectoryEntry 类

```csharp
public readonly record struct TableDirectoryEntry(string Tag, uint Checksum, uint Offset, uint Length)
{

}
```

如描述，以上的 Tag 属性就是由 4 个 ascii 组成的名称，不同的 Table 有不同的名称，这个名称有标准，详细请看水果家的文档： [Apple Advanced Typography Font Tables - TrueType Reference Manual - Apple Developer](https://developer.apple.com/fonts/TrueType-Reference-Manual/RM06/Chap6Tables.html)

对于二进制数据来说，指定一个数据大部分都会使用 Offset 偏移量和 Length 范围来进行指定。通过 TableDirectoryEntry 即可用来寻找 TTF 文件里面各个 Table 的信息

读取 TableDirectoryEntry 的方法如下

```csharp
    public static TableDirectoryEntry Read(BigEndianBinaryReader reader)
    {
        return new TableDirectoryEntry(reader.ReadAsciiString(4), reader.ReadUInt32(), reader.ReadUInt32(),
            reader.ReadUInt32());
    }
```

这里的 ReadAsciiString 是特别定义的方法，代码如下

```csharp
    public string ReadAsciiString(int charCount)
    {
        var buffer = ArrayPool<char>.Shared.Rent(charCount);
        for (int i = 0; i < charCount; i++)
        {
            buffer[i] = (char)(byte)Stream.ReadByte();
        }
        ArrayPool<char>.Shared.Return(buffer);
        return new string(buffer, 0, charCount);
    }
```

上面代码就是读取指定的字符数量拼接为字符串。定义这个方法是因为在 C# 里面，一个 char 是两个 byte 的大小。而在 TTF 里面，存放的是一个 byte 长度的 ascii 字符

如上文，由于多个 Table 的索引是连续的，可以连续读取。读取的数量通过 OffsetTable 的 NumTables 属性可以了解有多少个 Table 索引

```csharp
    public static TableDirectoryEntry[] Read(BigEndianBinaryReader reader, in OffsetTable offsetTable)
    {
        var tableDirectoryEntryArray = new TableDirectoryEntry[offsetTable.NumTables];

        for (int i = 0; i < tableDirectoryEntryArray.Length; i++)
        {
            tableDirectoryEntryArray[i] = TableDirectoryEntry.Read(reader);
        }

        return tableDirectoryEntryArray;
    }
```

在 TTF 字体，在 name 的 Table 存放了很多字符串信息，包括字体的字体名信息。例如黑体的英文名叫 simhei 而中文名 黑体

在 TTF 字体文件里面，根据字体 TTF 文件，可以读取出字体的字体名。一个字体可以有多个对应的字体名，接下来咱根据 TableDirectoryEntry 的信息，找到 name 这个 Table 接着读取出里面的字体名信息

在获取到 TTF 字体的所有 Table 索引的集合 `TableDirectoryEntry[]` 之后，即可通过其 Tag 找到名为 `name` 的 Table 的信息

```csharp
        using var bigEndianBinaryReader = new BigEndianBinaryReader(stream);
        var offsetTable = OffsetTable.Read(bigEndianBinaryReader);
        TableDirectoryEntry[] tableDirectoryEntryArray = TableDirectoryEntryArrayReader.Read(bigEndianBinaryReader, offsetTable);

        // [Glyph Properties Table - TrueType Reference Manual - Apple Developer](https://developer.apple.com/fonts/TrueType-Reference-Manual/RM06/Chap6prop.html )
        var nameDirTableEntry = tableDirectoryEntryArray.First(t => t.Tag == "name");
```

从以上的 `nameDirTableEntry` 可以拿到 name 这个 Table 所在 TTF 文件的绝对偏移和范围，从而可以进行读取。不同的 Table 有不同的存储定义，对于 NameTable 来说，定义的是包含了多个的 Name 记录，定义类型如下

```csharp
public readonly record struct NameTable(ushort Format, ushort Count, ushort StringOffset, NameRecord[] NameRecords)
{

}

public readonly record struct NameRecord(PlatformIdentifier PlatformId, ushort PlatformSpecificId, ushort LanguageId,
    NameIdentifier NameId, ushort Length, ushort Offset, string Value)
{

}
```

这里有一个坑在于，在 NameRecord 的 Value 属性上，定义的是一个字符串，字符串大家都知道这是一个可以使用带长度的不定长的类型。而在 TTF 里面，为了方便存储，就将字符串 Value 的数据定义和 NameRecord 的定义分开，将 Value 额外存放，需要通过 NameRecord 的 Offset 和 Length 属性进行读取。而另一个坑点就是在 NameRecord 定义的 Offset 属性不是 TTF 文件的绝对偏移量，而是一个相对于 NameTable 读取完成 NameRecord 集合的相对量。特别感谢 [Colby Newman](https://github.com/parzivail) 提供的 [https://github.com/parzivail/TtfLoader](https://github.com/parzivail/TtfLoader) 给了我读取的例子

先在 NameTable 读取出所有的 NameRecord 记录，再根据 NameRecord 读取出 Value 属性

```csharp
        var format = reader.ReadUInt16();
        var count = reader.ReadUInt16();
        var stringOffset = reader.ReadUInt16();

        var nameRecords = new NameRecord[count];
        for (int i = 0; i < count; i++)
        {
            nameRecords[i] = NameRecord.Read(reader);
        }
```

在 NameRecord 读取的方法如下

```csharp
public readonly record struct NameRecord(PlatformIdentifier PlatformId, ushort PlatformSpecificId, ushort LanguageId,
    NameIdentifier NameId, ushort Length, ushort Offset, string Value)
{
    public static NameRecord Read(BigEndianBinaryReader reader)
    {
        var platformId = (PlatformIdentifier) reader.ReadUInt16();
        var platformSpecificId = reader.ReadUInt16();
        var languageId = reader.ReadUInt16();
        var nameId = (NameIdentifier) reader.ReadUInt16();
        var length = reader.ReadUInt16();
        var offset = reader.ReadUInt16();

        // 这里的 Value 是在不连续的空间，推荐是先连续读取，然后再逐个 Value 获取
        // 先设置 Value 为 string.Empty 后续再读取。因为 Value 不是一个连续的值，需要根据 Offset 的内容读取
        return new NameRecord(platformId, platformSpecificId, languageId, nameId, length, offset, string.Empty);
    }
}
```

以上的 PlatformIdentifier 可以用来决定后续如何对字符串进行解码，大家都知道字符串有多个不同的二进制编码，如 UTF8 等不同的格式，根据 PlatformIdentifier 可以决定编码方式，定义如下

```csharp
namespace TtfReader;

public enum PlatformIdentifier : ushort
{
    Unicode = 0,
    Macintosh = 1,
    Reserved = 2,
    Microsoft = 3
}
```

以上的 NameIdentifier 表示的是这个 name 记录的内容的类型，例如此 name 记录的是字体名，还是版权信息等，定义如下

```csharp
namespace TtfReader;

public enum NameIdentifier : ushort
{
    CopyrightNotice = 0,
    FontFamily = 1,
    FontSubfamily = 2,
    UniqueSubfamilyId = 3,
    FullName = 4,
    NameTableVersion = 5,
    PostScriptName = 6,
    TrademarkNotice = 7,
    ManufacturerName = 8,
    DesignerName = 9,
    Description = 10,
    VendorUrl = 11,
    DesignerUrl = 12,
    LicenseDescription = 13,
    LicenseUrl = 14,
    PreferredFamily = 16,
    PreferredSubfamily = 17,
    CompatibleFull = 18,
    SampleText = 19,
}
```

本文读取的字体的字体名就是获取 FontFamily 类型

在 NameTable 读取完成 NameRecord 集合，就可以根据 NameRecord 的 Offset 等属性获取到字符串内容，这里的 Offset 相对的是读取完成集合之后的偏移而不是 TTF 的绝对值

```csharp
        // 连续的空间存放 NameRecord 对象，在 NameRecord 里面对应的字符串内容，是需要根据内容获取，放在不连续的空间
        for (int i = 0; i < count; i++)
        {
            var nameRecord = nameRecords[i];

            var buffer = ArrayPool<byte>.Shared.Rent(nameRecord.Length);

            // 这里的 Offset 是相对读取 NameRecord 集合完成的
            var currentPosition = reader.Stream.Position;
            reader.Stream.Seek(nameRecord.Offset, SeekOrigin.Current);
            var readCount = reader.Read(buffer, 0, nameRecord.Length);
            reader.Stream.Position = currentPosition;

            if (readCount != nameRecord.Length)
            {
                throw new EndOfStreamException();
            }

            switch (nameRecord.PlatformId)
            {
                case PlatformIdentifier.Unicode:
                case PlatformIdentifier.Microsoft:
                {
                    var value = Encoding.BigEndianUnicode.GetString(buffer, 0, nameRecord.Length);
                    nameRecord = nameRecord with { Value = value };
                    break;
                }
                case PlatformIdentifier.Macintosh:
                {
                    // Copy From https://github.com/parzivail/TtfLoader
                    if (nameRecord.PlatformSpecificId == 0)
                    {
                        var value = Encoding.ASCII.GetString(buffer, 0, nameRecord.Length);
                        nameRecord = nameRecord with { Value = value };
                    }

                    break;
                }
                case PlatformIdentifier.Reserved:
                    // 理论上不会进入
                    break;
                default:
                    throw new ArgumentOutOfRangeException();
            }

            ArrayPool<byte>.Shared.Return(buffer);
            nameRecords[i] = nameRecord;
        }
```

如此即可完成读取，只获取 FontFamily 的 NameIdentifier 进行输出即可输出字体定义的字体名

```csharp
foreach (var nameRecord in ttfInfo.NameTable.NameRecords)
{
    if (nameRecord.NameId == NameIdentifier.FontFamily)
    {
        Console.WriteLine(nameRecord.Value);
    }
}
```

以上就是完全自己写代码解析 TTF 文件，获取文件的字体名的方法。在字体里面，解析字体名是很简单的。在字体里面最难的就是获取每个字符的渲染信息，以及将字符进行绘制。对字符进行绘制只是做文本渲染里面最基础的一步，如果是想开发一个完全自己控制的文本库，那还需要比字符渲染更加复杂的排版规则。如果这个文本库期望做的稍微通用，能支持更多语言文化，那还需要考虑一下横竖排和合写字。本文只是学习目的自己解析 TTF 文件的文件名，代码没有达到项目可用，还请大家在实际项目使用时，仔细阅读官方文档，或者采用成熟的基础库，例如 WPF 的 FontFamily 类型

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/62c5e78042c7506734959a50cfc7ac1440182690/TtfReader) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/62c5e78042c7506734959a50cfc7ac1440182690/TtfReader) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 62c5e78042c7506734959a50cfc7ac1440182690
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 TtfReader 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。