# C# 16 进制字符串转 int 

最近在写硬件，发现有一些测试是做 16 进制的字符串，需要把他转换为整形才可以处理

本文告诉大家如何从 16 进制转整形

<!--more-->
<!-- CreateTime:2019/2/28 11:51:36 -->

<div id="toc"></div>
<!-- 标签：C# -->

如果输入的是 `0xaa` 这时转换 int 不能使用 `Parse` 不然会出现异常

```csharp
System.FormatException
```

如果需要转换十六进制就需要使用 Convert 才可以转换

```csharp
Convert.ToInt32("0xaa", 16)
```

使用这个方法才可以转换。实际使用这个方法转换不一定需要添加`0x`，直接使用`aa`也是可以

```csharp
Convert.ToInt32("0xaa", 16) == Convert.ToInt32("aa", 16)
```

我需要转换的是一个字符串，里面有很多数值，所以我就使用下面的方法。

如果输入的字符串是这个样子

```csharp
  var str =
                "AA BB CC 12 01 0D 00 34 38 34 35 32 30 41 35 33 46 37 30 2C 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 0A";
```

那么就需要先添加 `0x` 然后再转换，使用下面一句话可以转数组

```csharp
        var command = str.Split(' ').Select(temp => "0x" + temp).Select(temp => (byte) Convert.ToInt32(temp, 16))
                .ToArray();
```

这个方法大家可以直接拿去使用。

如果输入没有带`0x`那么另一个方法是 int.Parse ，因为这个方法可以设置如何转换，注意需要不带`0x`如果带了就出现异常

```csharp
int.Parse("aa", System.Globalization.NumberStyles.HexNumber)
```

[How to: Convert Between Hexadecimal Strings and Numeric Types ](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/how-to-convert-between-hexadecimal-strings-and-numeric-types )

![](https://i.loli.net/2018/08/19/5b78cf969217e.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
