
# dotnet OpenXML 读取形状轮廓线条样式序号超过主题样式列表数

在 OpenXML 中，默认的形状可以通过指定 LineReference 让形状使用文档主题里面的样式。文档主题里面包含多个样式，在形状里面指定样式通过的是序号的方法，如果在形状里面指定的序号超过了主题的数量，那么将会使用最后一项样式

<!--more-->


<!-- 发布 -->

开始之前，我准备了这份课件，我将课件和代码都放在 GitHub 上可以在本文最后找到链接

在这份课件中，第一页里面有一个形状元素，在形状元素里面定义了样式使用的是第 5 个样式

```xml
<p:sp>
 <p:style>
   <a:lnRef idx="5">
     <a:schemeClr val="accent1">
       <a:shade val="50000" />
     </a:schemeClr>
   </a:lnRef>
 </p:style>
</p:sp>
```

使用 [C# dotnet 使用 OpenXml 解析 PPT 文件](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-PPT-%E6%96%87%E4%BB%B6.html) 博客的方法打开这份课件，可以使用如下代码读取到使用的 LineReference 样式

```csharp
            using (var presentationDocument =
                DocumentFormat.OpenXml.Packaging.PresentationDocument.Open("测试.pptx", false))
            {
                var presentationPart = presentationDocument.PresentationPart;
                var slidePart = presentationPart.SlideParts.First();
                var shape = slidePart.Slide.Descendants<Shape>().First();
                var lineReference = shape.Descendants<LineReference>().First();
                /*
       <p:sp>
        <p:style>
          <a:lnRef idx="5">
            <a:schemeClr val="accent1">
              <a:shade val="50000" />
            </a:schemeClr>
          </a:lnRef>
        </p:style>
       </p:sp>
                */
                var lineStyle = lineReference.Index.Value;
                // 这里的值是 5 表示使用主题的第 5 个样式
                // 文档规定，Index是从1开始的
                // https://docs.microsoft.com/en-za/dotnet/api/documentformat.openxml.drawing.linereference?view=openxml-2.8.1
                lineStyle--;
            }
```

以上的细节是 `a:lnRef` 指定的 idx 是序号，而序号是从 1 开始的，咱的集合默认使用 0 开始

接下来是获取文档的主题，在 Office 的优先级是 Slide 然后是 SlideLayout 最后才是 SlideMaster 的主题

```csharp
                // 获取主题
                var themeOverride = slidePart.ThemeOverridePart?.ThemeOverride
                    ?? slidePart.SlideLayoutPart.ThemeOverridePart?.ThemeOverride;
                FormatScheme formatScheme = themeOverride?.FormatScheme;
                if (formatScheme is null)
                {
                    formatScheme = slidePart.SlideLayoutPart.SlideMasterPart.ThemePart.Theme.ThemeElements.FormatScheme;
                }
```

在这份课件，使用的是放在 Theme1.xml 里面的主题

```xml
                  <a:themeElements>
                    <a:fmtScheme name="Office">
                      <a:lnStyleLst>
                        <a:ln w="6350" cap="flat" cmpd="sng" algn="ctr">
                          <a:solidFill>
                            <a:schemeClr val="phClr" />
                          </a:solidFill>
                          <a:prstDash val="solid" />
                          <a:miter lim="800000" />
                        </a:ln>
                        <a:ln w="12700" cap="flat" cmpd="sng" algn="ctr">
                          <a:solidFill>
                            <a:schemeClr val="phClr" />
                          </a:solidFill>
                          <a:prstDash val="solid" />
                          <a:miter lim="800000" />
                        </a:ln>
                        <a:ln w="69050" cap="flat" cmpd="sng" algn="ctr">
                          <a:solidFill>
                            <a:srgbClr val="954F72" />
                          </a:solidFill>
                          <a:prstDash val="solid" />
                          <a:miter lim="800000" />
                        </a:ln>
                      </a:lnStyleLst>
                    </a:fmtScheme>
                  </a:themeElements>
```

以上的 FormatScheme 类就是存放 `a:fmtScheme` 的内容

使用下面代码获取线条样式

```csharp
                var lineStyleList = formatScheme.LineStyleList;
                var outlineList = lineStyleList.Elements<Outline>().ToList();
```

如果形状的样式序号没有大于主题定义的样式列表数量，那么使用对应的样式。如果定义的序号超过了主题定义的样式列表数量，就需要使用最后一个样式，请看代码

```csharp
                Outline themeOutline;
                if (lineStyle > outlineList.Count)
                {
                    themeOutline = outlineList[^1];
                }
                else
                {
                    themeOutline = outlineList[(int)lineStyle];
                }
```

上面代码获取的 Outline 就是形状线条在主题样式的值

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/36026db4de981dd8800a22b9e2aeaa9a174cb07a/LurkinurhuwarcuWhawhiweayea) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/36026db4de981dd8800a22b9e2aeaa9a174cb07a/LurkinurhuwarcuWhawhiweayea) 欢迎小伙伴访问

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。