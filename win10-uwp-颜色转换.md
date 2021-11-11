
# win10 uwp 颜色转换

本文告诉大家如何从字符串转颜色，从颜色转字符串

<!--more-->


<!-- CreateTime:2019/11/29 10:18:27 -->

<!-- csdn -->

## 字符串转颜色

在 WPF 可以使用下面的代码把十六进制的颜色字符串转颜色

```csharp
            Color color = (Color) ColorConverter.ConvertFromString("#FFDFD991");

```

```csharp
string hex = "#FFFFFF";  
Color color = System.Drawing.ColorTranslator.FromHtml(hex); 
```

但是 UWP 没这个方法，所以需要自己写一个方法

```csharp
        public SolidColorBrush GetSolidColorBrush(string hex)
        {
            hex = hex.Replace("#", string.Empty);
            byte a = (byte) (Convert.ToUInt32(hex.Substring(0, 2), 16));
            byte r = (byte) (Convert.ToUInt32(hex.Substring(2, 2), 16));
            byte g = (byte) (Convert.ToUInt32(hex.Substring(4, 2), 16));
            byte b = (byte) (Convert.ToUInt32(hex.Substring(6, 2), 16));
            return new SolidColorBrush(Windows.UI.Color.FromArgb(a, r, g, b));
        }
```

如果有小伙伴传入一个不带透明的，那么上面的代码就会出现异常，因为不带透明的颜色只有 6 个字符，所以就无法使用上面的代码，我修改了下面代码可以转换颜色

```csharp
       public SolidColorBrush GetSolidColorBrush(string hex)
        {
            hex = hex.Replace("#", string.Empty);

            bool existAlpha = hex.Length == 8;

            if (!existAlpha && hex.Length != 6)
            {
                throw new ArgumentException("输入的hex不是有效颜色");
            }

            int n = 0;
            byte a;
            if (existAlpha)
            {
                n = 2;
                a = (byte) ConvertHexToByte(hex, 0);
            }
            else
            {
                a = 0xFF;
            }

            var r = (byte) ConvertHexToByte(hex, n);
            var g = (byte) ConvertHexToByte(hex, n + 2);
            var b = (byte) ConvertHexToByte(hex, n + 4);
            return new SolidColorBrush(Windows.UI.Color.FromArgb(a, r, g, b));
        }

        private static uint ConvertHexToByte(string hex, int n)
        {
            return Convert.ToUInt32(hex.Substring(n, 2), 16);
        }
```

大家可以从上面代码发现 ConvertHexToByte 这就是 16 进制转 int 的方法，请看[C# 16 进制字符串转 int](https://blog.lindexi.com/post/C-16-%E8%BF%9B%E5%88%B6%E5%AD%97%E7%AC%A6%E4%B8%B2%E8%BD%AC-int.html )

但是存在这样写的颜色 `#FD92` `#DAC` 的颜色，所以还需要继续修改一下算法

```csharp
       public SolidColorBrush GetSolidColorBrush(string hex)
        {
            hex = hex.Replace("#", string.Empty);

            //#FFDFD991
            //#DFD991
            //#FD92
            //#DAC

            bool existAlpha = hex.Length == 8 || hex.Length == 4;
            bool isDoubleHex = hex.Length == 8 || hex.Length == 6;

            if (!existAlpha && hex.Length != 6 && hex.Length != 3)
            {
                throw new ArgumentException("输入的hex不是有效颜色");
            }

            int n = 0;
            byte a;
            int hexCount = isDoubleHex ? 2 : 1;
            if (existAlpha)
            {
                n = hexCount;
                a = (byte) ConvertHexToByte(hex, 0, hexCount);
                if (!isDoubleHex)
                {
                    a = (byte) (a * 16 + a);
                }
            }
            else
            {
                a = 0xFF;
            }

            var r = (byte) ConvertHexToByte(hex, n, hexCount);
            var g = (byte) ConvertHexToByte(hex, n + hexCount, hexCount);
            var b = (byte) ConvertHexToByte(hex, n + 2 * hexCount, hexCount);
            if (!isDoubleHex)
            {
                //#FD92 = #FFDD9922

                r = (byte) (r * 16 + r);
                g = (byte) (g * 16 + g);
                b = (byte) (b * 16 + b);
            }

            return new SolidColorBrush(Windows.UI.Color.FromArgb(a, r, g, b));
        }

        private static uint ConvertHexToByte(string hex, int n, int count = 2)
        {
            return Convert.ToUInt32(hex.Substring(n, count), 16);
        }
```

如果想看微软的转换，请看 https://referencesource.microsoft.com/#PresentationCore/Core/CSharp/System/Windows/Media/Parsers.cs

可以复制的源代码：

<script src="https://gist.github.com/lindexi/36c5e223ff77cfb8adc4909dec1576b5.js"></script>

如果你没有在上面看到代码，请点击 <https://gist.github.com/lindexi/36c5e223ff77cfb8adc4909dec1576b5 >

另外，如果有引用 WindowsCommunityToolkit 库，那么还有更简单的方法，就是通过 `Microsoft.Toolkit.Uwp.Helpers.ColorHelper.ToColor` 方法，这个方法提供了高性能的，很少的字符串分配的方法解析颜色，但是要求传入的颜色必须添加 `#` 开头

如何引用 WindowsCommunityToolkit 库？请通过 NuGet 安装 `Microsoft.Toolkit.Uwp` 库

另一个版本是尝试转换，也就是输入的字符串不符合预期的时候不会炸

```csharp
    static class ColorTextConverter
    {
        public static (bool success, Color? color) TryConvertSolidColorBrush(string hexColorText)
        {
            if (!hexColorText.StartsWith("#"))
            {
                return (false, null);
            }

            var hex = hexColorText.Replace("#", string.Empty);

            // 可以采用的格式如下
            // #FFDFD991   8 个字符 存在 Alpha 通道
            // #DFD991     6 个字符
            // #FD92       4 个字符 存在 Alpha 通道
            // #DAC        3 个字符

            bool existAlpha = hex.Length == 8 || hex.Length == 4;
            bool isDoubleHex = hex.Length == 8 || hex.Length == 6;

            if (!existAlpha && hex.Length != 6 && hex.Length != 3)
            {
                return (false, null);
            }

            int n = 0;
            byte a;
            // 表示十六进制的字符数量，使用两个字符还是一个字符表示
            int hexCount = isDoubleHex ? 2 : 1;
            if (existAlpha)
            {
                n = hexCount;
                var convertHexToByteResult = ConvertHexToByte(hex, 0, hexCount);
                if (!convertHexToByteResult.success)
                {
                    return (false, null);
                }

                a = convertHexToByteResult.hexNumber;
                if (!isDoubleHex)
                {
                    a = (byte) (a * 16 + a);
                }
            }
            else
            {
                a = 0xFF;
            }

            var result = ConvertHexToByte(hex, n, hexCount);
            if (!result.success)
            {
                return (false, null);
            }
            var r = result.hexNumber;

            result = ConvertHexToByte(hex, n + hexCount, hexCount);
            if (!result.success)
            {
                return (false, null);
            }
            var g = result.hexNumber;

            result = ConvertHexToByte(hex, n + 2 * hexCount, hexCount);
            if (!result.success)
            {
                return (false, null);
            }
            var b = result.hexNumber;

            if (!isDoubleHex)
            {
                //#FD92 = #FFDD9922

                r = (byte) (r * 16 + r);
                g = (byte) (g * 16 + g);
                b = (byte) (b * 16 + b);
            }

            return (true, Color.FromArgb(a, r, g, b));
        }

        private static (bool success, byte hexNumber) ConvertHexToByte(string hex, int n, int count = 2)
        {
            var hexText = hex.Substring(n, count);

            if (byte.TryParse(hexText, NumberStyles.HexNumber, new NumberFormatInfo(), out var hexNumber))
            {
                return (true, hexNumber);
            }

            return (false, byte.MinValue);
        }
    }
```

以上都有使用 Substring 方法从而需要分配内存，下面提供一个无多余内存分配的方法

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

## 颜色转字符串

如果需要从颜色转字符串是很简单

```csharp
Color.ToString()
```

上面的代码就可以输出字符串

![](https://i.loli.net/2018/04/08/5aca000c4b395.jpg)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。