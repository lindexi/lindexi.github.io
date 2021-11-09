
# dotnet OpenXML 读取 PPT 内嵌 xlsx 格式 Excel 表格的信息

在 Office 中，可以在 PPT 里面插入表格，插入表格有好多不同的方法，对应 OpenXML 文档存储的更多不同的方式。本文来介绍如何读取 PPT 内嵌 xlsx 格式的 xls+ 表格的方法

<!--more-->



<!-- 发布 -->

读取方法和 [dotnet OpenXML 读取 PPT 内嵌 ole 格式 Excel 表格的信息](https://blog.lindexi.com/post/dotnet-OpenXML-%E8%AF%BB%E5%8F%96-PPT-%E5%86%85%E5%B5%8C-ole-%E6%A0%BC%E5%BC%8F-Excel-%E8%A1%A8%E6%A0%BC%E7%9A%84%E4%BF%A1%E6%81%AF.html ) 差不多，对于 Office 2019 以上版本，插入 Excel 表格用的不是 OLE 文件的方式，而是放入一个 xlsx 文件

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

插入的 rId3 的资源对应的内容如下

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">
  <Relationship Id="rId3" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/package" Target="../embeddings/Microsoft_Excel_Worksheet.xlsx" />
</Relationships>
```

也就是说插入到页面的对应的 xlsx 文件存放路径如下

```
ppt\embeddings\Microsoft_Excel_Worksheet.xlsx
```

和读取 OLE 的 xls+ 方式不同的在于不需要读取 OLE 文件拿到 xlsx 文件，只需要通过 Part 读取即可。通过如上代码可以看到在 Slide 页面存放的代码几乎相同，需要加上一点判断逻辑，才能决定是从 Part 读取还是从 OLE 文件读取

通过判断 `part.ContentType` 是 `"application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"` 的内容，即可了解是嵌入 xlsx 文件，而不是 ole 格式文件

读取的逻辑如下

```csharp
            using var presentationDocument = PresentationDocument.Open(file.FullName, false);
            var slide = presentationDocument.PresentationPart!.SlideParts.First().Slide;

            var graphicFrame = slide.CommonSlideData!.ShapeTree!.GetFirstChild<GraphicFrame>()!;
            var graphic = graphicFrame.Graphic!;
            var graphicData = graphic.GraphicData!;
            var alternateContent = graphicData.GetFirstChild<AlternateContent>()!;
            var choice = alternateContent.GetFirstChild<AlternateContentChoice>()!;
            var oleObject = choice.GetFirstChild<OleObject>()!;
            Debug.Assert(oleObject.GetFirstChild<OleObjectEmbed>() != null);
            var id = oleObject.Id!;
            var part = slide.SlidePart!.GetPartById(id!);
            Debug.Assert(part.ContentType == "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
```

再加上判断当前的 Graphic 期望使用的应用

```csharp
            // 预期字符串是 “Excel.Sheet.12” 等内容
            var isEmbedExcel = oleObject.ProgId?.Value?.StartsWith("Excel.Sheet", StringComparison.OrdinalIgnoreCase) is true;

            Debug.Assert(isEmbedExcel);
```

将 Part 读取放入到本地文件，用于后续解析 Xlsx 文件。为什么不能通过 `part.GetStream` 的方式，对返回的 Stream 进行读取即可？原因是此 Stream 是不支持随机访问的，这个 Stream 是从 System.IO.Packaging 拿到的，为了解决 N 多的坑，设计为不支持随机读取，只能顺序读取。而在解析 Xlsx 时，需要进行随机读取，否则就需要将整个文件内容都加载到内存，为了减少内存的占用，存放到文件

```csharp
            var tempFolder = @"F:\temp";
            if (!Directory.Exists(tempFolder))
            {
                tempFolder = System.IO.Path.GetTempPath();
            }

            var xlsxFile = System.IO.Path.Combine(tempFolder, System.IO.Path.GetRandomFileName() + ".xlsx");
            using (var fileStream = File.OpenWrite(xlsxFile))
            {
                using var partStream = part.GetStream(FileMode.Open,FileAccess.Read);
                partStream.CopyTo(fileStream);
            }
```

后续就是读取 xlsx 的逻辑

```csharp
            using var spreadsheetDocument = SpreadsheetDocument.Open(xlsxFile, false);
            var sheets = spreadsheetDocument.WorkbookPart!.Workbook.Sheets;
```

更多读取 Excel 的方法请看 [C# dotnet WPF 使用 OpenXml 解析 Excel 文件](https://blog.lindexi.com/post/C-dotnet-WPF-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-Excel-%E6%96%87%E4%BB%B6.html)

本文以上的测试文件和代码放在[github](https://github.com/lindexi/lindexi_gd/tree/75c6c17055d7c254e648b7c1836f0657c72fc77a/Pptx) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/75c6c17055d7c254e648b7c1836f0657c72fc77a/Pptx) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 75c6c17055d7c254e648b7c1836f0657c72fc77a
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





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。