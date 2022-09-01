# WPF 从 RGB 字符串转纯色画刷的方法

本文告诉大家几个方法用来从 RGB 字符串转纯色的 SolidColorBrush 画刷

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

在 Windows 下，约定的编程规范里，颜色的 RGB 的字符串表示方法是 `#[A]RGB` 的格式，一定是 R 红色，接着是 G 绿色，最后是 B 蓝色，其中可选首个 Alpha 通道

颜色格式如下

- #AARRGGBB: 这是最完全的字符串表示方式
- #RRGGBB: 省略了 Alpha 通道，此表示方式的 Alpha 通道等同于 0xFF 的值，表示不透明的纯色
- #ARGB: 对于 #AARRGGBB 不同的是，只使用一个字符表示一个通道，例如 `#AC12` 等同于 `#AACC1122` 的颜色
- #RGB: 和 #ARGB 差不多，只是省略 Alpha 通道，表示不透明的纯色

在开始进行转换时，如果发现转换的颜色不符合预期，还请先仔细阅读一下传入的颜色字符串，看字符串的格式是否符合预期

## BrushConverter

使用框架自带的 BrushConverter 进行转换的方法如下：

先有一个 BrushConverter 对象，此对象可以被重复使用。这个 BrushConverter 类型也是 XAML 里面转换颜色字符串所采用的转换器

```csharp
            var brushConverter = new BrushConverter();
```

使用 BrushConverter 的 ConvertFrom 方法即可转换为纯色画刷。只不过 BrushConverter 的 ConvertFrom 方法是 TypeConverter 定义的，返回值是 object 类型，需要进行转换

```csharp
            var solidColorBrush = (SolidColorBrush) brushConverter.ConvertFrom("#CCFF00");
```

如此即可完成转换

## 手动解析

如果不想使用框架自带的，也可以进行手动转换颜色，以下是我从 [win10 uwp 颜色转换](https://blog.lindexi.com/post/win10-uwp-%E9%A2%9C%E8%89%B2%E8%BD%AC%E6%8D%A2.html ) 拷贝的代码

```csharp
static (bool success, byte a, byte r, byte g, byte b) ConvertToColor(string input)
{
    bool startWithPoundSign = input.StartsWith('#');
    var colorStringLength = input.Length;
    if (startWithPoundSign) colorStringLength -= 1;
    int currentOffset = startWithPoundSign ? 1 : 0;
    // 可以采用的格式如下
    // #FFDFD991   8 个字符 存在 Alpha 通道
    // #DFD991     6 个字符
    // #FD92       4 个字符 存在 Alpha 通道
    // #DAC        3 个字符
    if (colorStringLength == 8
        || colorStringLength == 6
        || colorStringLength == 4
        || colorStringLength == 3)
    {
        bool success;
        byte result;
        byte a;

        int readCount;
        // #DFD991     6 个字符
        // #FFDFD991   8 个字符 存在 Alpha 通道
        //if (colorStringLength == 8 || colorStringLength == 6)
        if (colorStringLength > 5)
        {
            readCount = 2;
        }
        else
        {
            readCount = 1;
        }

        bool includeAlphaChannel = colorStringLength == 8 || colorStringLength == 4;

        if (includeAlphaChannel)
        {
            (success, result) = HexCharToNumber(input, currentOffset, readCount);
            if (!success) return default;
            a = result;
            currentOffset += readCount;
        }
        else
        {
            a = 0xFF;
        }

        (success, result) = HexCharToNumber(input, currentOffset, readCount);
        if (!success) return default;
        byte r = result;
        currentOffset += readCount;

        (success, result) = HexCharToNumber(input, currentOffset, readCount);
        if (!success) return default;
        byte g = result;
        currentOffset += readCount;

        (success, result) = HexCharToNumber(input, currentOffset, readCount);
        if (!success) return default;
        byte b = result;

        return (true, a, r, g, b);
    }

    return default;
}

static (bool success, byte result) HexCharToNumber(string input, int offset, int readCount)
{
    Debug.Assert(readCount == 1 || readCount == 2, "要求 readCount 只能是 1 或者 2 的值，这是框架限制，因此不做判断");

    byte result = 0;

    for (int i = 0; i < readCount; i++, offset++)
    {
        var c = input[offset];
        byte n;
        if (c >= '0' && c <= '9')
        {
            n = (byte)(c - '0');
        }
        else if (c >= 'a' && c <= 'f')
        {
            n = (byte)(c - 'a' + 10);
        }
        else if (c >= 'A' && c <= 'F')
        {
            n = (byte)(c - 'A' + 10);
        }
        else
        {
            return default;
        }

        result *= 16;
        result += n;
    }

    if (readCount == 1)
    {
        result = (byte)(result * 16 + result);
    }

    return (true, result);
}
```