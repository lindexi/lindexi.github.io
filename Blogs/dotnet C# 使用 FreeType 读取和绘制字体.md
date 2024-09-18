---
title: dotnet C# 使用 FreeType 读取和绘制字体
description: 本文将和大家介绍在 C# 里面简单使用 SharpFont 对 FreeType 的封装，读取 ttf 等字体文件信息，绘制出某个文字到图片文件

<!--more-->

tags: dotnet C#
category: 
---

<!-- CreateTime:2024/04/19 20:31:36 -->

<!-- 发布 -->
<!-- 博客 -->

由于本文使用的 SharpFont 库已经很久没有维护了，本文的例子里面使用的 .NET 框架就退回到 .NET Framework 4.7.2 版本。我大概看了代码，预计 dotnet 6 等版本还是能够兼容的，只是为了方便我写例子代码，减少遇到一些奇怪的问题，本文的例子就采用比较旧的框架

开始之前先感谢 [Robert Rouhani](https://github.com/Robmaister) 大佬开源的 <https://github.com/Robmaister/SharpFont> 项目，尽管这个项目已经很久没有维护了

按照 .NET 的惯例，先通过 NuGet 安装库，我通过编辑 csproj 文件快速进行安装，编辑之后的 csproj 项目文件的代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net472</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="SharpFont" Version="4.0.1" />
  </ItemGroup>
</Project>

```

先通过 SetDllDirectory 按照 x64 或 x86 方式加载库，代码如下，以下这部分感觉是基础库没有封装好的部分

```csharp

        public static void Main(string[] args)
        {
            var folderPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
            var nugetFolder = Path.Combine(folderPath, @"..\.nuget\packages\sharpfont.dependencies");
            // 如果自己的 nuget 没有设置为其他路径的话
            var sharpFontDependenciesNuGetFolder = Directory.EnumerateDirectories(nugetFolder).First();

            if (Environment.Is64BitProcess)
            {
                var libraryFolder = Path.Combine(sharpFontDependenciesNuGetFolder, @"bin\msvc12\x64\");
                SetDllDirectory(libraryFolder);
            }
            else
            {
                var libraryFolder = Path.Combine(sharpFontDependenciesNuGetFolder, @"bin\msvc12\x86\");
                SetDllDirectory(libraryFolder);
            }

        }

        [DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)]
        private static extern bool SetDllDirectory(string path);
```

以上代码我是去找 NuGet 文件夹里面的依赖包里面的文件

完成以上步骤之后，即可创建出 Face 对象。如以下代码随意给一个字体文件进行测试

```csharp
            var library = new Library();
            var face = new Face(library, @"C:\windows\fonts\simfang.ttf");
```

接下来的代码将演示如何获取某个字符在字体里面的信息，以及将这个字体用这个字体渲染到本地图片文件

获取字符在字体里面的信息，需要先获取到字符在字体里面的索引，代码如下

```csharp
            uint glyphIndex = face.GetCharIndex('林');
```

以上代码就可以获取到 `林` 字在字体文件里面的索引

接下来为了将字体加载到 slot 里面，需要先设置一点必要的初始化参数

```csharp
            // 设置字体大小，修复 SharpFont.FreeTypeException:“FreeType error: Invalid size handle.”
            face.SetCharSize(26,0,96,0);
```

接着将字体加载到 slot 里面，用于后续获取 Glyph 属性，获取信息

```csharp
            // 加载 slot 用于后续渲染
            face.LoadGlyph(glyphIndex, LoadFlags.Default, LoadTarget.Normal);
```

完成以上步骤即可使用以下代码，获取到字符的信息

```csharp
            float advanceX = (float) face.Glyph.Advance.X; // same as the advance in metrics
            float bearingX = (float) face.Glyph.Metrics.HorizontalBearingX;
            float width = face.Glyph.Metrics.Width.ToSingle();
            float glyphTop = (float) face.Glyph.Metrics.HorizontalBearingY;
            float glyphBottom = (float) (face.Glyph.Metrics.Height - face.Glyph.Metrics.HorizontalBearingY);
```

以上的各个变量就是对于传入的字符的信息

将字体渲染到图片需要借助 GDI 部分的辅助，先调用 RenderGlyph 方法，再通过 ToGdipBitmap 转换为 System.Drawing.Bitmap 对象，用于保存到本地文件

```csharp
            face.Glyph.RenderGlyph(RenderMode.Normal);

            face.Glyph.Bitmap.ToGdipBitmap().Save("1.png");
```

以上的代码我都放在一个 Main 方法里面，代码如下

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;
using SharpFont;

namespace ChewukeriLudikanal
{
    internal class Program
    {
        public static void Main(string[] args)
        {
            var folderPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
            var nugetFolder = Path.Combine(folderPath, @"..\.nuget\packages\sharpfont.dependencies");
            // 如果自己的 nuget 没有设置为其他路径的话
            var sharpFontDependenciesNuGetFolder = Directory.EnumerateDirectories(nugetFolder).First();

            if (Environment.Is64BitProcess)
            {
                var libraryFolder = Path.Combine(sharpFontDependenciesNuGetFolder, @"bin\msvc12\x64\");
                SetDllDirectory(libraryFolder);
            }
            else
            {
                var libraryFolder = Path.Combine(sharpFontDependenciesNuGetFolder, @"bin\msvc12\x86\");
                SetDllDirectory(libraryFolder);
            }

            var library = new Library();
            var face = new Face(library, @"C:\windows\fonts\simfang.ttf");

            uint glyphIndex = face.GetCharIndex('林');

            // 设置字体大小，修复 SharpFont.FreeTypeException:“FreeType error: Invalid size handle.”
            face.SetCharSize(26,0,96,0);

            // 加载 slot 用于后续渲染
            face.LoadGlyph(glyphIndex, LoadFlags.Default, LoadTarget.Normal);

            // 获取字体信息
            float advanceX = (float) face.Glyph.Advance.X; // same as the advance in metrics
            float bearingX = (float) face.Glyph.Metrics.HorizontalBearingX;
            float width = face.Glyph.Metrics.Width.ToSingle();
            float glyphTop = (float) face.Glyph.Metrics.HorizontalBearingY;
            float glyphBottom = (float) (face.Glyph.Metrics.Height - face.Glyph.Metrics.HorizontalBearingY);

            // 尝试获取字间距
            //kern = (float) face.GetKerning(glyphIndex, face.GetCharIndex(cNext), KerningMode.Default).X;
            face.Glyph.RenderGlyph(RenderMode.Normal);

            face.Glyph.Bitmap.ToGdipBitmap().Save("1.png");
        }

        [DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)]
        private static extern bool SetDllDirectory(string path);
    }
}
```

尝试运行代码，可以看到运行之后输出了 1.png 文件，用图片查看器打开可以看到里面绘制出了字符

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/df6a50e5af79104064e91aca92f72d331fac7161/ChewukeriLudikanal) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/df6a50e5af79104064e91aca92f72d331fac7161/ChewukeriLudikanal) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin df6a50e5af79104064e91aca92f72d331fac7161
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin df6a50e5af79104064e91aca92f72d331fac7161
```

获取代码之后，进入 ChewukeriLudikanal 文件夹，即可获取到源代码

其他字体相关请参阅：

- [dotnet 解析 TTF 字体文件格式](https://blog.lindexi.com/post代码占用的空/dotnet-%E8%A7%A3%E6%9E%90-TTF-%E5%AD%97%E4%BD%93%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F.html )

- [WPF 从文件加载字体](https://blog.lindexi.com/post/WPF-%E4%BB%8E%E6%96%87%E4%BB%B6%E5%8A%A0%E8%BD%BD%E5%AD%97%E4%BD%93.html )
