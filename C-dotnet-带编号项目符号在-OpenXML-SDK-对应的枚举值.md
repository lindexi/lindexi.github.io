
# C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值

本文告诉大家在 OpenXML SDK 里面文本框的文本带自动编号的项目符号，不同的编号在 OpenXML SDK 上的枚举值

<!--more-->


<!-- CreateTime:2020/3/26 17:08:16 -->



在 OpenXML SDK 使用 [TextAutoNumberSchemeValues](https://docs.microsoft.com/zh-cn/dotnet/api/documentformat.openxml.drawing.textautonumberschemevalues) 枚举表示自动编号的项目符号使用的自动编号是哪个

以下是一些常用的枚举值

```
ArabicPeriod  1.2.3.
ArabicParenR  1)2)3)
RomanUpperCharacterPeriod  I. II. III.
RomanLowerCharacterPeriod  i. ii. iii.
AlphaUpperCharacterPeriod  A. B. C.
AlphaLowerCharacterParenR  a) b) c)
AlphaLowerCharacterPeriod  a. b. c.
CircleNumberDoubleBytePlain  ① ② ③
EastAsianJapaneseDoubleBytePeriod  一. 二.
```

## ArabicPeriod

<!-- ![](image/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值0.png) -->
![](http://cdn.lindexi.site/lindexi%2F2020326172152923.jpg)

## RomanUpperCharacterPeriod

<!-- ![](image/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值1.png) -->

![](http://cdn.lindexi.site/lindexi%2F20203261721428156.jpg)

## AlphaUpperCharacterPeriod

<!-- ![](image/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值3.png) -->

![](http://cdn.lindexi.site/lindexi%2F2020326172266980.jpg)

## AlphaLowerCharacterParenR

<!-- ![](image/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值4.png) -->

![](http://cdn.lindexi.site/lindexi%2F2020326172231169.jpg)

## CircleNumberDoubleBytePlain

<!-- ![](image/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值6.png) -->

![](http://cdn.lindexi.site/lindexi%2F20203261729295719.jpg)

## EastAsianJapaneseDoubleBytePeriod

<!-- ![](image/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值7.png) -->

![](http://cdn.lindexi.site/lindexi%2F20203261729517413.jpg)

## AlphaLowerCharacterPeriod

<!-- ![](image/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值/C# dotnet 带编号项目符号在 OpenXML SDK 对应的枚举值5.png) -->

![](http://cdn.lindexi.site/lindexi%2F20203261722596311.jpg)

这就是 PPT 页面上的常用的枚举值

更多细节请看 ECMA-376 的 20.1.10.61 章文档

## 所有可用值

以下是我整理的枚举中文注释值

```csharp
public enum TextAutoNumberSchemeValues
{
    /// <summary>
    /// 带双括号的小写字母，如 (a)、 (b)、 (c)
    /// </summary>
    AlphaLowerCharacterParenBoth,

    /// <summary>
    /// 带双括号的大写字母，如 (A)、 (B)、 (C)
    /// </summary>
    AlphaUpperCharacterParenBoth,

    /// <summary>
    /// 带右括号的小写字母，如 a)、 b)、 c)
    /// </summary>
    AlphaLowerCharacterParenR,

    /// <summary>
    /// 带右括号的大写字母，如 A)、 B)、 C)
    /// </summary>
    AlphaUpperCharacterParenR,

    /// <summary>
    /// 带点号的小写字母，如 a.、 b.、 c.
    /// </summary>
    AlphaLowerCharacterPeriod,

    /// <summary>
    /// 带点号的大写字母，如 A.、 B.、 C.
    /// </summary>
    AlphaUpperCharacterPeriod,

    /// <summary>
    /// 带双括号的阿拉伯数字，如 (1)、 (2)、 (3)
    /// </summary>
    ArabicParenBoth,

    /// <summary>
    /// 带右括号的阿拉伯数字，如 1)、 2)、 3)
    /// </summary>
    ArabicParenR,

    /// <summary>
    /// 带点号的阿拉伯数字，如 1.、 2.、 3.
    /// </summary>
    ArabicPeriod,

    /// <summary>
    /// 不带任何符号的阿拉伯数字，如 1、 2、 3
    /// </summary>
    ArabicPlain,

    /// <summary>
    /// 带双括号的小写罗马数字，如 (i)、 (ii)、 (iii)
    /// </summary>
    RomanLowerCharacterParenBoth,

    /// <summary>
    /// 带双括号的大写罗马数字，如 (I)、 (II)、 (III)
    /// </summary>
    RomanUpperCharacterParenBoth,

    /// <summary>
    /// 带右括号的小写罗马数字，如 i)、 ii)、 iii)
    /// </summary>
    RomanLowerCharacterParenR,

    /// <summary>
    /// 带右括号的大写罗马数字，如 I)、 II)、 III)
    /// </summary>
    RomanUpperCharacterParenR,

    /// <summary>
    /// 带点号的小写罗马数字，如 i.、 ii.、 iii.
    /// </summary>
    RomanLowerCharacterPeriod,

    /// <summary>
    /// 带点号的大写罗马数字，如 I.、 II.、 III.
    /// </summary>
    RomanUpperCharacterPeriod,

    /// <summary>
    /// 带圈的双字节阿拉伯数字，如 ①、 ②、 ③
    /// </summary>
    CircleNumberDoubleBytePlain,

    /// <summary>
    /// 黑底的圈的双字节阿拉伯数字，如 ①、 ②、 ③
    /// </summary>
    CircleNumberWingdingsBlackPlain,

    /// <summary>
    /// 白底的圈的双字节阿拉伯数字，如 ①、 ②、 ③
    /// </summary>
    CircleNumberWingdingsWhitePlain,

    /// <summary>
    /// 带点号的双字节阿拉伯数字，如 1.、 2.、 3.
    /// </summary>
    ArabicDoubleBytePeriod,

    /// <summary>
    /// 不带任何符号的双字节阿拉伯数字，如 1、 2、 3
    /// </summary>
    ArabicDoubleBytePlain,

    /// <summary>
    /// 带点的简体中文数字，如 一.、 二.、 三.
    /// </summary>
    EastAsianSimplifiedChinesePeriod,

    /// <summary>
    /// 不带任何符号的简体中文数字，如 一、 二、 三
    /// </summary>
    EastAsianSimplifiedChinesePlain,

    /// <summary>
    /// 带点的繁体中文数字，如 一.、 二.、 三.
    /// </summary>
    EastAsianTraditionalChinesePeriod,

    /// <summary>
    /// 不带任何符号的繁体中文数字，如 一、 二、 三
    /// </summary>
    EastAsianTraditionalChinesePlain,

    /// <summary>
    /// 带点的日文双字节数字，如 一.、 二.、 三.
    /// </summary>
    EastAsianJapaneseDoubleBytePeriod,

    /// <summary>
    /// 不带任何符号的日韩数字，如 一、 二、 三
    /// </summary>
    EastAsianJapaneseKoreanPlain,

    /// <summary>
    /// 带点的日韩数字，如 一.、 二.、 三.
    /// </summary>
    EastAsianJapaneseKoreanPeriod,

    /// <summary>
    /// 阿拉伯语1
    /// </summary>
    Arabic1Minus,

    /// <summary>
    /// 阿拉伯语2
    /// </summary>
    Arabic2Minus,

    /// <summary>
    /// 希伯来语2
    /// </summary>
    Hebrew2Minus,

    /// <summary>
    /// 带点的泰语
    /// </summary>
    ThaiAlphaPeriod,

    /// <summary>
    /// 带右括号的泰语
    /// </summary>
    ThaiAlphaParenthesisRight,

    /// <summary>
    /// 带双括号的泰语
    /// </summary>
    ThaiAlphaParenthesisBoth,

    /// <summary>
    /// 带点的泰语数字
    /// </summary>
    ThaiNumberPeriod,

    /// <summary>
    /// 带右括号的泰语数字
    /// </summary>
    ThaiNumberParenthesisRight,

    /// <summary>
    /// 带双括号的泰语数字
    /// </summary>
    ThaiNumberParenthesisBoth,

    /// <summary>
    /// 带点的印地语字母
    /// </summary>
    HindiAlphaPeriod,

    /// <summary>
    /// 带点的印地语数字
    /// </summary>
    HindiNumPeriod,

    /// <summary>
    /// 带右括号的印地语数字
    /// </summary>
    HindiNumberParenthesisRight,

    /// <summary>
    /// 带点的印地语
    /// </summary>
    HindiAlpha1Period,
}
```

我使用代码列举的所有可用值，制作成了一份 PPTX 文档，将其放在 GitHub 上，详细请看：<https://github.com/lindexi/lindexi_gd/blob/03e507c006ff11196b44970a148f6f36d5a88ad1/Pptx/ChiwewibaiKucajochall/%E7%BC%96%E5%8F%B7%E9%A1%B9%E7%9B%AE%E7%AC%A6%E5%8F%B7%E6%95%B4%E7%90%86%E5%A4%A7%E5%85%A8.pptx>

其生成代码也放在 [github](https://github.com/lindexi/lindexi_gd/tree/03e507c006ff11196b44970a148f6f36d5a88ad1/Pptx/ChiwewibaiKucajochall) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/03e507c006ff11196b44970a148f6f36d5a88ad1/Pptx/ChiwewibaiKucajochall) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 03e507c006ff11196b44970a148f6f36d5a88ad1
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 03e507c006ff11196b44970a148f6f36d5a88ad1
```

获取代码之后，进入 Pptx/ChiwewibaiKucajochall 文件夹，即可获取到源代码




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。