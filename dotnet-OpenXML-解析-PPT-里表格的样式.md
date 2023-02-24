
# dotnet OpenXML 解析 PPT 里表格的样式

在 PPT 里面的表格可以通过表格样式配置决定表格的样式，本文将和大家介绍如何获取和解析表格的样式

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

本文属于 OpenXML 系列博客，有一定的上下文，详细请参阅 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

在 PPT 里面的表格，如存放在页面 Slide 里面的表格，可以通过 `a:tableStyleId` 属性存放表格的样式 Id 值。表格的样式可以采用自定义表格样式，也可以采用应用自带的样式。为了兼容性，大部分情况下，即使采用应用自带的样式，也是会将样式模版放入到 TableStylesPart 里面去，也就是对应的 TableStyles.xml 文件里面

放在 Slide 里面的表格的代码大概如下

```xml
      <p:graphicFrame>
        <p:nvGraphicFramePr>
          <p:cNvPr id="4" name="表格 4">
            ...
          </p:cNvPr>
          <p:nvPr>
           ...
          </p:nvPr>
        </p:nvGraphicFramePr>
        <p:xfrm>
          <a:off x="2032000" y="719666" />
          <a:ext cx="8127999" cy="1112520" />
        </p:xfrm>
        <a:graphic>
          <a:graphicData uri="http://schemas.openxmlformats.org/drawingml/2006/table">
            <a:tbl>
              <a:tblPr firstRow="1" bandRow="1">
                <a:tableStyleId>{21E4AEA4-8DFA-4A89-87EB-49C32662AFE0}</a:tableStyleId>
              </a:tblPr>
              <a:tblGrid>
                ...
              </a:tblGrid>
              ...
            </a:tbl>
          </a:graphicData>
        </a:graphic>
      </p:graphicFrame>
```

以上的 `<a:tableStyleId>{21E4AEA4-8DFA-4A89-87EB-49C32662AFE0}</a:tableStyleId>` 就是用来定制表格采用哪个样式

对应的样式的细节定义，大部分时候可以从 TableStylesPart 里面，也就是对应的 TableStyles.xml 文件里面去找到，里面存放的样式代码大概如下

```xml
  <a:tblStyleLst xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main" def="{5C22544A-7EE6-4342-B048-85BDC9FD1C3A}">
    <a:tblStyle styleId="{21E4AEA4-8DFA-4A89-87EB-49C32662AFE0}">
      <a:wholeTbl>
        ...
      </a:wholeTbl>
    </a:tblStyle>
  </a:tblStyleLst>
```

另一些样式是直接存放到 TableProperties 也就是对应的 `a:tblPr` 的 TableStyle 里面去，对于这样定义的文档来说，直接获取 TableStyle 即可

大概的获取代码如下

```csharp
// 先已拿到 Table 对象，也就是 a:tbl 属性
// 以下代码存放在获取表格样式方法

            var tableProperties = table.TableProperties;

            if (tableProperties is null)
            {
                return null;
            }

            // 有一些PPT的样式是放在tblPr属性里的。
            var tableStyle = tableProperties.GetFirstChild<TableStyle>();
            if (tableStyle != null)
            {
                return tableStyle;
            }

            // 2016新版本PPT是通过TableStyleId去寻找样式表的。
            var tableStyleId = tableProperties.GetFirstChild<TableStyleId>();
            if (tableStyleId is null)
            {
                return null;
            }

            var text = tableStyleId.Text;
            // 以下的 document 是 PresentationDocument 类型
            var tableStyleList = document.PresentationPart.TableStylesPart
                ?.TableStyleList;
            var tableStyleEntry = tableStyleList?.Elements<TableStyleEntry>()
                .FirstOrDefault(temp => temp.StyleId == text);
```

对于一些文档来说，如果采用的是应用自带的样式，可能没有将样式内容存放到 TableStylesPart 里面去。此时就采用应用级样式，所谓应用级样式属性，那就是编写在代码里面的样式。比如本文所使用 `{21E4AEA4-8DFA-4A89-87EB-49C32662AFE0}` 样式。此样式是可以在 PPTX 文档里面默认不写样式定义的

对应以上的代码，也就是无法从 TableStyleList 里面获取到 `tableStyleEntry` 对象。此时就需要采用代码自己编写预设的样式

```csharp
            // 从 Application 级获取表格的样式。
            tableStyleEntry ??= TableStyleEntryProvider.CreateTableStyleEntry(text);

            return tableStyleEntry;
```

这里的 TableStyleEntryProvider 类型是我编写的代码，里面包含了大量的预设表格样式。此代码放入到 [dotnetCampus.DocumentFormat.OpenXml.Flatten](https://www.nuget.org/packages/dotnetCampus.DocumentFormat.OpenXml.Flatten) 库里面，代码在 GitHub 完全开源，详细请参阅 [https://github.com/dotnet-campus/DocumentFormat.OpenXml.Extensions](https://github.com/dotnet-campus/DocumentFormat.OpenXml.Extensions)

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。