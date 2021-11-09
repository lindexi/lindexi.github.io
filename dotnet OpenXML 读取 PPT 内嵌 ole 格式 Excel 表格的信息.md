# dotnet OpenXML 读取 PPT 内嵌 ole 格式 Excel 表格的信息

在 Office 中，可以在 PPT 里面插入表格，插入表格有好多不同的方法，对应 OpenXML 文档存储的更多不同的方式。本文来介绍如何读取 PPT 内嵌 ole 格式的 xls+ 表格的方法

<!--more-->
<!-- CreateTime:2021/9/2 8:52:29 -->

<!-- 发布 -->

在 Office 的 PPT 中，插入表格可以对应多个不同的方式：

- 通过 GraphicData 内嵌到 PPTX 页面里面
- 通过嵌入文件方式
- 通过 SmartArt 模拟的表格，本质上就是 SmartArt 元素

其中通过嵌入文件方式可以分为以下不同的嵌入方式：

- 通过外嵌 Microsoft_Excel_Worksheet.xlsx 格式，此格式可以解析。这是在 Office 2019 的默认
- 通过外嵌 oleObject1.bin 格式，此格式是 ole 格式，里面包含 xls+ 格式
- 通过外嵌 oleObject1.bin 格式，此格式是 ole 格式，里面包含了 xls 格式

什么是 xls+ 格式？其实这个名字我没有找到权威的文档来说明。大概是在 Office 2016 的默认行为是如此，点击表格，插入 Excel 电子表格时嵌入的文档就是此格式。这个格式存放方式是 ole 格式，在此 OLE 文件里面，将存放 OpenXML 格式的 xlsx 格式的表格文件，以下将详细告诉大家此格式

在 Slide.xml 页面里面，存放的是在 GraphicFrame 下的内容，简化的 OpenXML 文档如下

```xml
      <p:graphicFrame>
        <p:nvGraphicFramePr>
          <p:cNvPr id="9" name="表格 1" />
        </p:nvGraphicFramePr>
        <p:xfrm>
          <a:off x="5405438" y="3241675" />
          <a:ext cx="3438525" cy="2009775" />
        </p:xfrm>
        <a:graphic>
          <a:graphicData uri="http://schemas.openxmlformats.org/presentationml/2006/ole">
            <mc:AlternateContent xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006">
              <mc:Choice xmlns:v="urn:schemas-microsoft-com:vml" Requires="v">
                <p:oleObj spid="_x0000_s1026" name="工作表" r:id="rId3" imgW="3438630" imgH="2009788" progId="Excel.Sheet.12">
                  <p:embed />
                </p:oleObj>
              </mc:Choice>
              <mc:Fallback>
               <!-- 忽略 -->
              </mc:Fallback>
            </mc:AlternateContent>
          </a:graphicData>
        </a:graphic>
      </p:graphicFrame>
```

以上逻辑核心的就是存放的嵌入的 oleObj 对象，可以在 Slide.xml.rels 文件里面找到如下定义内容

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">
  <Relationship Id="rId3" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/oleObject" Target="../embeddings/oleObject1.bin" />
</Relationships>
```

也就是说嵌入的表格是放在 embeddings 文件夹下的 oleObject1.bin 文件，这是一个 OLE 文件。本质上来说 OLE 和 ZIP 等压缩格式是同等级的，是用来做存储的，也就是说 OLE 格式本身不是特定给 Excel 表格使用的，仅仅只是用来做存储而已。大家是否还记得 ppt 和 pptx 的差别，上古（2003）的时候，采用的是格式是 ppt 格式，此格式的存储就是 OLE 存储方式，也可以这样认为，古时候的 xls 和 ppt 等都是 OLE 文件。但是新版本的 pptx 和 xlsx 等都是 OpenXML 格式

嵌入到 PPT 的 oleObject1.bin 也就是 OLE 文件，对应上古的格式。但是有一些不同的是，此文件不属于 xls 文件格式，而是细分为两个类别，其中一个是在 OLE 里面存放 xls 的，另一个存放的是 xlsx 的。也就是说需要将 oleObject1.bin 展开，才可以获取里面的表格文件。本文将在 OLE 里面存放 xlsx 格式的嵌入方式称为 xls+ 格式

先来开始从 OpenXML 文档读取到 OLE 嵌入文件的逻辑

和通用的 PPTX 文件解析相同的逻辑，先读取文件，我的测试文件在首页就嵌入了表格。本文所有的代码和测试文件都可以在本文末尾找到下载方式

```csharp
            var file = new FileInfo("Test.pptx");

            using var presentationDocument = PresentationDocument.Open(file.FullName, false);
            var slide = presentationDocument.PresentationPart!.SlideParts.First().Slide;
```

接下来获取 GraphicFrame 和里层的信息

```csharp
            var graphicFrame = slide.CommonSlideData!.ShapeTree!.GetFirstChild<GraphicFrame>()!;
            var graphic = graphicFrame.Graphic!;
            var graphicData = graphic.GraphicData!;
```

如上述文档，在 GraphicData 里面存放的是 AlternateContent 元素，此元素里面再嵌入 OLE 文件

```csharp
            var alternateContent = graphicData.GetFirstChild<AlternateContent>()!;
            var choice = alternateContent.GetFirstChild<AlternateContentChoice>()!;
            var oleObject = choice.GetFirstChild<OleObject>()!;
            Debug.Assert(oleObject.GetFirstChild<OleObjectEmbed>() != null);
```

通过以上逻辑即可获取到对应的 OleObject 对象。本文上面的例子代码仅仅只是用于本文的测试文件，对于其他文件不确定是否存在表格的，还请自行判断空，而不是采用本文的断言方式。本文的例子里的代码为了清晰，就不添加其他分支判断

以上代码拿到了 OleObject 即可获取到对应的 oleObject1.bin 文件。在 OpenXML SDK 里面，不会真的将 PPTX 文件解压缩，原因有两个：第一个是性能考虑，第二个是有一些内容解压缩之后会丢失信息（不是使用文件存放的，只是兼容zip格式而已）而导致了尝试使用路径读取 oleObject1.bin 文件是不可行的。通过 [dotnet OpenXML 为什么资源使用 Relationship 引用](https://blog.lindexi.com/post/dotnet-OpenXML-%E4%B8%BA%E4%BB%80%E4%B9%88%E8%B5%84%E6%BA%90%E4%BD%BF%E7%94%A8-Relationship-%E5%BC%95%E7%94%A8.html ) 博客了解到，读取方法如下

```csharp
            var id = oleObject.Id!;
            var part = slide.SlidePart!.GetPartById(id!);
            Debug.Assert(part.ContentType== "application/vnd.openxmlformats-officedocument.oleObject");
```

使用 `part.GetStream(FileMode.Open)` 就可以打开 oleObject1.bin 对应的 Stream 对象

然而这是一个 OLE 对象，为了解析此文件，咱需要引入一个基于 MPL 协议（宽松，可商业，无须开源）的 [Open MCDF](https://github.com/ironfede/openmcdf) 库，这是一个完全由 C# 实现的读取 OLE 格式文档的库，在我做 VisualStudio 插件时也用到，请看 [dotnet Roslyn 通过读取 suo 文件获取解决方案的启动项目](https://blog.lindexi.com/post/dotnet-Roslyn-%E9%80%9A%E8%BF%87%E8%AF%BB%E5%8F%96-suo-%E6%96%87%E4%BB%B6%E8%8E%B7%E5%8F%96%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E7%9A%84%E5%90%AF%E5%8A%A8%E9%A1%B9%E7%9B%AE.html )

在 csproj 上添加如下代码进行安装 [Open MCDF](https://github.com/ironfede/openmcdf) 库

```xml
  <PackageReference Include="OpenMcdf" Version="2.2.1.9" />
```

当前的 csproj 项目文件代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net5.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

  <ItemGroup>
    <None Update="Test.pptx">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="dotnetCampus.OpenXmlUnitConverter" Version="1.7.1" />
    <PackageReference Include="DocumentFormat.OpenXml" Version="2.13.1" />
    <PackageReference Include="OpenMcdf" Version="2.2.1.9" />
  </ItemGroup>

</Project>
```

尽管在 [Open MCDF](https://github.com/ironfede/openmcdf) 库提供了 CompoundFile 的构造函数可以传入 Stream 对象，但是因为在 OpenXML 的 Part 取出的 Stream 是不可随机访问的（为了解决 N 多的坑，在 System.IO.Packaging 限制）因此以下代码是不可用的

```csharp
 var compoundFile = new CompoundFile(part.GetStream(FileMode.Open));
```

执行上面代码将会提示 `OpenMcdf.CFException:“Cannot load a non-seekable Stream”` 而失败

为了使用 [Open MCDF](https://github.com/ironfede/openmcdf) 库读取，需要先存放到本地文件，代码如下

```csharp
            var tempFolder = System.IO.Path.GetTempPath();

            var oleFile = System.IO.Path.Combine(tempFolder, System.IO.Path.GetRandomFileName());
            using (var fileStream = File.OpenWrite(oleFile))
            {
                using var stream = part.GetStream(FileMode.Open);
                stream.CopyTo(fileStream);
            }
```

打开此 OLE 文件代码如下

```csharp
            var compoundFile = new CompoundFile(oleFile);
```

从此 OLE 文件读取出 xlsx 文件的代码如下

```csharp
            var packageStream = compoundFile.RootStorage.GetStream("Package");
            var xlsxFile = System.IO.Path.Combine(tempFolder, System.IO.Path.GetRandomFileName()+".xlsx");
            using (var fileStream = File.OpenWrite(xlsxFile))
            {
                fileStream.Write(packageStream.GetData().AsSpan());
            }
```

在获取到 xlsxFile 文件之后，即可进行 Excel 解析，读取里面的信息

```csharp
            using var spreadsheetDocument = SpreadsheetDocument.Open(xlsxFile,false);
            var sheets = spreadsheetDocument.WorkbookPart!.Workbook.Sheets;
```

更多读取 Excel 的方法请看 [C# dotnet WPF 使用 OpenXml 解析 Excel 文件](https://blog.lindexi.com/post/C-dotnet-WPF-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-Excel-%E6%96%87%E4%BB%B6.html)

本文不再详细告诉大家如何读取此 Excel 内容

本文以上的测试文件和代码放在[github](https://github.com/lindexi/lindexi_gd/tree/976b039620120286bed59eda5363a87b592941ca/Pptx) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/976b039620120286bed59eda5363a87b592941ca/Pptx) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 976b039620120286bed59eda5363a87b592941ca
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 Pptx 文件夹

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

更多参考：

- [MS-OFFDI].pdf
- [MS-XLS].pdf
- [MS-OI 29500].pdf

我嫌弃 [Open MCDF](https://github.com/ironfede/openmcdf) 库占用的内存太大，于是写了一个只读的版本，此版本能够使用尽可能少的内存，代码正在提回官方，请看 [Add the readonly version by lindexi · Pull Request #87 · ironfede/openmcdf](https://github.com/ironfede/openmcdf/pull/87 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。