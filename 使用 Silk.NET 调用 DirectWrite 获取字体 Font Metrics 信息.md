# 使用 Silk.NET 调用 DirectWrite 获取字体 Font Metrics 信息

本文将告诉大家在 dotnet 里面，通过 Silk.NET 库调用 DirectWrite 获取给定字体的 Font Metrics 信息

<!--more-->
<!-- CreateTime:2025/07/24 07:12:00 -->

<!-- 发布 -->
<!-- 博客 -->

使用 dotnet 基金会里的 Silk.NET 库可以执行许多渲染层的调用，如 DirectX 或 OpenGL 等。整个 Silk.NET 库设计上都是非常高性能的，极大量使用了函数指针。使用起来需要触碰不安全代码，调用损耗接近可以忽略。属于一个使用门槛难度高，但性能上限极高的库

本文也属于我的渲染系列博客，更多渲染相关博客请参阅 [渲染相关](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

按照 dotnet 的惯例，在使用之前，先通过 NuGet 安装库。按照 Silk.NET 的设计，将 DirectWrite 相关的辅助类都放在了 Silk.NET.Direct2D 库里面。可通过编辑 csproj 项目文件快速完成安装，编辑之后的 csproj 项目文件的代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Silk.NET.Direct2D" Version="2.22.0" />
  </ItemGroup>

</Project>
```

在以上 csproj 项目文件代码里面，通过 `<AllowUnsafeBlocks>true</AllowUnsafeBlocks>` 开启不安全代码

开始之前先来看看 FontMetrics 结构体的定义，定义代码如下。对应着 DirectWrite 的 DWRITE_FONT_METRICS 结构体

```csharp
    public struct FontMetrics
    {
        public ushort DesignUnitsPerEm;

        public ushort Ascent;

        public ushort Descent;

        public short LineGap;

        public ushort CapHeight;

        public ushort XHeight;

        public short UnderlinePosition;

        public ushort UnderlineThickness;

        public short StrikethroughPosition;

        public ushort StrikethroughThickness;
    }
```

各个字段的含义说明如下

- DesignUnitsPerEm：字体每个 em 单位对应的设计单位数量。字体文件使用自己的设计单位坐标系，em 方框是用于调整和对齐字形的参考正方形，此属性决定了该坐标系的缩放基准。
- Ascent：字体上升线高度（以设计单位计）。表示从字体字符对齐框顶部到英文基线的距离。
- Descent：字体下降线高度（以设计单位计）。表示从字体字符对齐框底部到英文基线的距离。
- LineGap：建议行间距（以设计单位计）。推荐在行之间添加的额外空白，以提升可读性。推荐的行距为 Ascent + Descent + LineGap，该值通常为零或正数，但也可能为负数。
- CapHeight：大写字母高度（以设计单位计）。表示从英文基线到典型大写字母顶部（如“H”）的距离。
- XHeight：小写字母“x”高度（以设计单位计）。表示从英文基线到小写“x”或类似字符顶部的距离。
- UnderlinePosition：下划线位置（以设计单位计）。表示下划线相对于英文基线的位置，通常为负值，以便下划线位于基线下方。
- UnderlineThickness：建议下划线粗细（以设计单位计）。
- StrikethroughPosition：删除线位置（以设计单位计）。表示删除线相对于英文基线的位置，通常为正值，以便删除线位于基线上方。
- StrikethroughThickness：建议删除线粗细（以设计单位计）。

本文将演示如何将传入的宋体字体文件，通过 DirectWrite 进行解析，从而获取到字体的 FontMetrics 信息

按照 Silk.NET 的设计，先调用 `DWrite.GetApi()` 方法获取 DWrite 辅助类对象，代码如下

```csharp
DWrite dWrite = DWrite.GetApi();
```

接着开始创建 DirectWrite 工厂。以下所有相关的基础设施都将从 DirectWrite 工厂创建，代码如下

```csharp
ComPtr<IDWriteFactory6> factory = dWrite.DWriteCreateFactory<IDWriteFactory6>(FactoryType.Shared);
```

为了方便演示，我这里使用了系统自带的宋体文件，代码如下

```csharp
// 宋体字体
var fontFile = @"C:\windows\fonts\simsun.ttc";
```

本文内容里面只给出关键代码片段，如需要全部的项目文件，可到本文末尾找到本文所有代码的下载方法

先建立字体文件的引用，代码如下

```csharp
unsafe
{
    HResult hr = 0;

    IDWriteFontFaceReference* fontFaceReference;

    fixed (char* pFontFile = fontFile)
    {
        hr = factory.Handle->CreateFontFaceReference(pFontFile, null, (uint)0, FontSimulations.None,
            &fontFaceReference);
        hr.Throw();
    }

}
```

以上代码中的 HResult 是一个辅助结构体，如果调用的方法返回的 HResult 包含错误信息，则 `HResult.Throw` 方法将抛出异常。如果返回不包含错误信息，则啥都不发生

以上代码传入的获取一个非加粗、非斜体的 `FontSimulations.None` 正常字体样式，获取其首个字体作为引用。传入的字体文件可能是一个字族，以上代码只获取其首个

在这里也可以看到 Silk.NET 库的封装非常底层，调用都通过指针。这样的缺点很明显，那就是难写，且写错了程序会炸。优点是提供了足够高的控制性，依靠开发人员自身强大的技术能力，在高控制的辅助下编写高性能的程序

拿到 IDWriteFontFaceReference 对象，即可调用 CreateFontFace 获取 IDWriteFontFace3 对象，代码如下

```csharp
    IDWriteFontFace3* fontFace3;
    fontFaceReference->CreateFontFace(&fontFace3);
```

获取到 IDWriteFontFace3 对象，即可通过其 GetMetrics 方法获取字体信息，代码如下

```csharp
    FontMetrics fontMetrics = default;
    fontFace3->GetMetrics(&fontMetrics);
```

此时就可以愉快输出 FontMetrics 结构体的内容了，如以下代码

```csharp
    Console.WriteLine($"Ascent: {fontMetrics.Ascent}");
    Console.WriteLine($"Descent: {fontMetrics.Descent}");
    Console.WriteLine($"LineGap: {fontMetrics.LineGap}");
    Console.WriteLine($"CapHeight: {fontMetrics.CapHeight}");
    Console.WriteLine($"XHeight: {fontMetrics.XHeight}");
    Console.WriteLine($"DesignUnitsPerEm: {fontMetrics.DesignUnitsPerEm}");
```

字体里面还包含了字体名，为了更好在控制台输出，我这里还编写了额外的代码获取字体名信息，代码如下

```csharp
    fontFace3->GetFamilyNames(&dWriteLocalizedStrings);
    PrintLocalizedStrings(dWriteLocalizedStrings);

unsafe void PrintLocalizedStrings(IDWriteLocalizedStrings* dWriteLocalizedStrings)
{
    uint count = dWriteLocalizedStrings->GetCount();
    List<(string LocaleName, string Name)> list = new((int)count);

    for (uint i = 0; i < count; i++)
    {
        uint length = 0;
        dWriteLocalizedStrings->GetLocaleNameLength(i, &length);

        // 加一解决 \0 的问题
        length += 1;
        char* localeNameBuffer = stackalloc char[(int)length];
        dWriteLocalizedStrings->GetLocaleName(i, localeNameBuffer, length);

        // zh-cn 等输出
        string localeName = new string(localeNameBuffer, 0, (int)length - 1);

        dWriteLocalizedStrings->GetStringLength(i, &length);
        length += 1;
        char* nameBuffer = stackalloc char[(int)length];
        dWriteLocalizedStrings->GetString(i, nameBuffer, length);
        string name = new string(nameBuffer, 0, (int)length - 1);

        list.Add((localeName, name));
    }

    foreach (var (localeName, name) in list)
    {
        if (localeName == "zh-cn")
        {
            Console.WriteLine($"FontName: {name}");
            return;
        }
    }

    Console.WriteLine(list.FirstOrDefault().Name);
}
```

尝试运行代码，大概可见以下输出内容

```
FontName: 宋体
Ascent: 220
Descent: 36
LineGap: 36
CapHeight: 175
XHeight: 116
DesignUnitsPerEm: 256
LineSpacing: 292 1.140625
```

最后一行是行距信息，是根据 LineGap 的说明内容进行计算的，官方文档的说明如下

> The line gap in font design units. Recommended additional white space to add between lines to improve legibility. The recommended line spacing (baseline-to-baseline distance) is the sum of ascent, descent, and lineGap. The line gap is usually positive or zero but can be negative, in which case the recommended line spacing is less than the height of the character alignment box.

计算代码如下，计算之后能够获取到和 WPF 的 `System.Windows.Media.FontFamily` 的 LineSpacing 属性相同的值

```csharp
    var lineSpacing = fontMetrics.Ascent + fontMetrics.Descent + fontMetrics.LineGap;
    Console.WriteLine($"LineSpacing: {lineSpacing} {lineSpacing / (double)fontMetrics.DesignUnitsPerEm}");
```

通过除以 DesignUnitsPerEm 可以获取比例值。大部分情况下都是取比例值参与字号进行计算，字体本身不带具体的单位，如像素单位这些。更正确来说，只有执行渲染的时候，在 UI 框架里面提字体的像素值才有意义

整个 Program.cs 代码如下

```csharp
using Silk.NET.Core.Native;
using Silk.NET.DirectWrite;

DWrite dWrite = DWrite.GetApi();
ComPtr<IDWriteFactory6> factory = dWrite.DWriteCreateFactory<IDWriteFactory6>(FactoryType.Shared);

// 宋体字体
var fontFile = @"C:\windows\fonts\simsun.ttc";

unsafe
{
    HResult hr = 0;

    IDWriteFontFaceReference* fontFaceReference;

    fixed (char* pFontFile = fontFile)
    {
        hr = factory.Handle->CreateFontFaceReference(pFontFile, null, (uint)0, FontSimulations.None,
            &fontFaceReference);
        hr.Throw();
    }

    IDWriteFontFace3* fontFace3;
    fontFaceReference->CreateFontFace(&fontFace3);

    FontMetrics fontMetrics = default;
    fontFace3->GetMetrics(&fontMetrics);

    IDWriteLocalizedStrings* dWriteLocalizedStrings;

    fontFace3->GetFamilyNames(&dWriteLocalizedStrings);
    PrintLocalizedStrings(dWriteLocalizedStrings);

    Console.WriteLine($"Ascent: {fontMetrics.Ascent}");
    Console.WriteLine($"Descent: {fontMetrics.Descent}");
    Console.WriteLine($"LineGap: {fontMetrics.LineGap}");
    Console.WriteLine($"CapHeight: {fontMetrics.CapHeight}");
    Console.WriteLine($"XHeight: {fontMetrics.XHeight}");
    Console.WriteLine($"DesignUnitsPerEm: {fontMetrics.DesignUnitsPerEm}");

    var lineSpacing = fontMetrics.Ascent + fontMetrics.Descent + fontMetrics.LineGap;
    Console.WriteLine($"LineSpacing: {lineSpacing} {lineSpacing / (double)fontMetrics.DesignUnitsPerEm}");
}

unsafe void PrintLocalizedStrings(IDWriteLocalizedStrings* dWriteLocalizedStrings)
{
    uint count = dWriteLocalizedStrings->GetCount();
    List<(string LocaleName, string Name)> list = new((int)count);

    for (uint i = 0; i < count; i++)
    {
        uint length = 0;
        dWriteLocalizedStrings->GetLocaleNameLength(i, &length);

        // 加一解决 \0 的问题
        length += 1;
        char* localeNameBuffer = stackalloc char[(int)length];
        dWriteLocalizedStrings->GetLocaleName(i, localeNameBuffer, length);

        // zh-cn 等输出
        string localeName = new string(localeNameBuffer, 0, (int)length - 1);

        dWriteLocalizedStrings->GetStringLength(i, &length);
        length += 1;
        char* nameBuffer = stackalloc char[(int)length];
        dWriteLocalizedStrings->GetString(i, nameBuffer, length);
        string name = new string(nameBuffer, 0, (int)length - 1);

        list.Add((localeName, name));
    }

    foreach (var (localeName, name) in list)
    {
        if (localeName == "zh-cn")
        {
            Console.WriteLine($"FontName: {name}");
            return;
        }
    }

    Console.WriteLine(list.FirstOrDefault().Name);
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/ab6143f859aabb5d83c45160df57d2f116231475/DirectX/DWrite/LeejefelearlaceKujercuhawyearke) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/ab6143f859aabb5d83c45160df57d2f116231475/DirectX/DWrite/LeejefelearlaceKujercuhawyearke) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin ab6143f859aabb5d83c45160df57d2f116231475
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin ab6143f859aabb5d83c45160df57d2f116231475
```

获取代码之后，进入 DirectX/DWrite/LeejefelearlaceKujercuhawyearke 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )