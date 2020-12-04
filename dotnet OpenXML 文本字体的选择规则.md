# dotnet OpenXML 文本字体的选择规则

在 Office 的文本排版里面，会根据字符选择使用哪个字体插槽。也就是实际上在 Office 里面可以在一个文本段里面指定多个字体，会根据实际的字符使用不同的字体

<!--more-->
<!-- CreateTime:7/8/2020 4:40:01 PM -->



在做 Office 解析的时候，在 OpenXML SDK 里面是没有找到表示字体的属性的，只能找到 LatinFont 和 EastAsianFont 和 ComplexScriptFont 和 SymbolFont 这几个

```csharp
        public TextFontType LatinFont { get; set; } // latin
        public TextFontType EastAsianFont { get; set; } // ea
        public TextFontType ComplexScriptFont { get; set; } // cs
        public TextFontType SymbolFont { get; set; } // sym
```

而这每个里面都可以使用不同的字体，如下面的文档

```xml
<a:r>
   <a:rPr …>
     <a:cs typeface="Times New Roman"/>
     <a:latin typeface="songti"/>
   </a:rPr>
   <a:t>
 العربية 
   </a:t> 
</a:r>
```

可以看到这里包含了 `a:cs` 和 `a:latin` 分别表示 LatinFont 和 ComplexScriptFont 两个不同的插槽

那么具体的这个我也看不懂的文本应该使用 `Times New Roman` 还是使用宋体字体，这就需要使用下面这个表格的内容了

在 Office 里面将会根据字符的 Unicode 决定使用哪个字体插槽的字体

- U+0000–U+007F latin font 
- U+0080–U+00A6 latin font 
- U+00A9–U+00AF latin font 
- U+00B2–U+00B3 latin font 
- U+00B5–U+00D6 latin font 
- U+00D8–U+00F6 latin font 
- U+00F8–U+058F latin font 
- U+0590–U+074F cs font 
- U+0780–U+07BF cs font 
- U+0900–U+109F cs font 
- U+10A0–U+10FF latin font 
- U+1200–U+137F latin font 
- U+13A0–U+177F latin font 
- U+1D00–U+1D7F latin font 
- U+1E00–U+1FFF latin font 
- U+1780–U+18AF cs font 
- U+2000–U+200B latin font 
- U+200C–U+200F cs font 
- U+2010–U+2029 latin font
- U+2018–U+201F ea font
- U+202A–U+202F cs font 
- U+2030–U+2046 latin font 
- U+204A–U+245F latin font 
- U+2670–U+2671 cs font 
- U+27C0–U+2BFF latin font 
- U+3099–U+309A ea font  
- U+D835        latin font
- U+F000–U+F0FF sym font 
- U+FB00–U+FB17 latin font 
- U+FB1D–U+FB4F cs font 
- U+FE50–U+FE6F latin font
- U+1D400–U+1D7FF latin font 
- 其他 ea font

<!-- (U\+[\dA-F]*\SU\+[\dA-F]*) -->

因此这和 [C# dotnet 获取某个字符所在 Unicode 字符平面映射](https://blog.lindexi.com/post/C-dotnet-%E8%8E%B7%E5%8F%96%E6%9F%90%E4%B8%AA%E5%AD%97%E7%AC%A6%E6%89%80%E5%9C%A8-Unicode-%E5%AD%97%E7%AC%A6%E5%B9%B3%E9%9D%A2%E6%98%A0%E5%B0%84.html) 不相同

这是 [ECMA-376](http://www.ecma-international.org/publications/standards/Ecma-376.htm ) 规定的

小伙伴如果不想自动动手写代码， 可以参考我在 WPF 项目里面使用的代码，理论上你在 dotnet 的项目里面，能使用 C# 7.0 的语法就能使用

这是上层的使用方法，这里的 text 是输入的字符串

```csharp
for (var i = 0; i < text.Length; i++)
{
   var c = text[i];

   TextType currentType = CharUnicodeRangeTextFontType.GetFontLang(c);
}
```

先定义一个枚举，表示当前的文本是什么，请看代码

```csharp
        public enum TextType
        {
            /// <summary>
            /// 默认的文本 中文系 东亚字符
            /// </summary>
            EastAsian,
            /// <summary>
            /// 拉丁英文系
            /// </summary>
            Latin,
            /// <summary>
            /// 复杂脚本
            /// </summary>
            ComplexScript,

            /// <summary>
            /// 特殊符号
            /// </summary>
            Symbol,
        }
```

添加一个辅助类

```csharp
   /// <summary>
    /// 文本内容范围判断类
    /// </summary>
    public class TextRangePattern
    {
        /// <summary>
        /// 创建文本内容范围判断类
        /// </summary>
        /// <param name="minChar"></param>
        /// <param name="maxChar"></param>
        public TextRangePattern(int minChar, int maxChar)
            : this((char)minChar, (char)maxChar)
        {

        }

        /// <summary>
        /// 创建文本内容范围判断类
        /// </summary>
        /// <param name="minChar"></param>
        /// <param name="maxChar"></param>
        public TextRangePattern(char minChar, char maxChar)
        {
            MinChar = minChar;
            MaxChar = maxChar;
        }

        /// <summary>
        /// 最小字符
        /// </summary>
        public char MinChar { get; }

        /// <summary>
        /// 最大字符
        /// </summary>
        public char MaxChar { get; }

        /// <summary>
        /// 是否输入的字符在范围内
        /// </summary>
        /// <param name="c"></param>
        /// <returns></returns>
        public bool IsInRange(char c)
        {
            return !(c < MinChar || c > MaxChar);
        }
    }
```

接着就是创建 CharUnicodeRangeTextFontType 辅助类

```csharp
    static class CharUnicodeRangeTextFontType
    {
        // 如果觉得下面的很多单词不知道是什么意思，请看 https://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%B9%B3%E9%9D%A2%E6%98%A0%E5%B0%84

        private static TextRangePattern[] LatinFontTextRangePatternList { get; } =
        {
            // - U\+([\dA-F]+)–U\+([\dA-F]+).*
            new TextRangePattern(0x0000, 0x007F), // - U+0000–U+007F latin font 
            new TextRangePattern(0x0080, 0x00A6), // - U+0080–U+00A6 latin font 
            new TextRangePattern(0x00A9, 0x00AF), // - U+00A9–U+00AF latin font 
            new TextRangePattern(0x00B2, 0x00B3), // - U+00B2–U+00B3 latin font 
            new TextRangePattern(0x00B5, 0x00D6), // - U+00B5–U+00D6 latin font 
            new TextRangePattern(0x00D8, 0x00F6), // - U+00D8–U+00F6 latin font 
            new TextRangePattern(0x00F8, 0x058F), // - U+00F8–U+058F latin font 
            new TextRangePattern(0x10A0, 0x10FF), // - U+10A0–U+10FF latin font 
            new TextRangePattern(0x1200, 0x137F), // - U+1200–U+137F latin font 
            new TextRangePattern(0x13A0, 0x177F), // - U+13A0–U+177F latin font 
            new TextRangePattern(0x1D00, 0x1D7F), // - U+1D00–U+1D7F latin font 
            new TextRangePattern(0x1E00, 0x1FFF), // - U+1E00–U+1FFF latin font 
            new TextRangePattern(0x2000, 0x200B), // - U+2000–U+200B latin font 
            new TextRangePattern(0x2010, 0x2029), // - U+2010–U+2029 latin font
            new TextRangePattern(0x2030, 0x2046), // - U+2030–U+2046 latin font 
            new TextRangePattern(0x204A, 0x245F), // - U+204A–U+245F latin font 
            new TextRangePattern(0x27C0, 0x2BFF), // - U+27C0–U+2BFF latin font 
            new TextRangePattern(0xFB00, 0xFB17), // - U+FB00–U+FB17 latin font 
            new TextRangePattern(0xFE50, 0xFE6F), // - U+FE50–U+FE6F latin font
            new TextRangePattern(0xD835, 0xD835), // - U+D835        latin font
        };

        private static TextRangePattern[] ComplexScriptFontTextRangePatternList { get; } =
        {
            new TextRangePattern(0x0590, 0x074F), // - U+0590–U+074F cs font 
            new TextRangePattern(0x0780, 0x07BF), // - U+0780–U+07BF cs font 
            new TextRangePattern(0x0900, 0x109F), // - U+0900–U+109F cs font 
            new TextRangePattern(0x1780, 0x18AF), // - U+1780–U+18AF cs font 
            new TextRangePattern(0x200C, 0x200F), // - U+200C–U+200F cs font 
            new TextRangePattern(0x202A, 0x202F), // - U+202A–U+202F cs font 
            new TextRangePattern(0x2670, 0x2671), // - U+2670–U+2671 cs font 
            new TextRangePattern(0xFB1D, 0xFB4F), // - U+FB1D–U+FB4F cs font 
        };

        private static TextRangePattern[] EastAsianFontTextRangePatternList { get; } =
        {
            new TextRangePattern(0x2018, 0x201F), // - U+2018–U+201F ea font
            new TextRangePattern(0x3099, 0x309A), // - U+3099–U+309A ea font  
        };

        private static TextRangePattern[] SymbolFontTextRangePatternList { get; } =
        {
            new TextRangePattern(0xF000, 0xF0FF), // - U+F000–U+F0FF sym font 
        };

        private static (TextType textType, TextRangePattern[] fontTextRangePatternList)[] PatternList { get; } =
        {
            (TextType.Latin, LatinFontTextRangePatternList),
            (TextType.ComplexScript, ComplexScriptFontTextRangePatternList),
            (TextType.EastAsian, EastAsianFontTextRangePatternList),
            (TextType.Symbol, SymbolFontTextRangePatternList)
        };

        /// <summary>
        /// 根据传入的字符判断当前是哪个语言项
        /// </summary>
        /// <param name="ch"></param>
        /// <returns></returns>
        public static TextType GetFontLang(char ch)
        {
            // 按照 [dotnet OpenXML 文本字体的选择规则](https://blog.lindexi.com/post/dotnet-OpenXML-%E6%96%87%E6%9C%AC%E5%AD%97%E4%BD%93%E7%9A%84%E9%80%89%E6%8B%A9%E8%A7%84%E5%88%99.html)

            foreach (var (textType, fontTextRangePatternList) in PatternList)
            {
                if (fontTextRangePatternList.Any(temp => temp.IsInRange(ch)))
                {
                    return textType;
                }
            }

            // - 其他 ea font
            return TextType.EastAsian;
        }
    }
```

上面代码忽略 utf16 的字符
  
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
