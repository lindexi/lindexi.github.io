# dotnet OpenXML 幻灯片 PPTX 的 Slide Id 和页面序号的关系

在使用 OpenXML SDK 进行 Office 文档的解析时，对幻灯片 PPTX 文档的页面解析也许会遇到页面顺序的问题，本文告诉大家在 Office 文档里面页面的序号和顺序之间的关系以及如何读取页面序号

<!--more-->
<!-- CreateTime:2020/10/15 11:42:20 -->




在开始之前，我期望你是了解一些 PPT 解析的相关知识的，入门级博客请看 [C# dotnet 使用 OpenXml 解析 PPT 文件](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-PPT-%E6%96%87%E4%BB%B6.html)

更多博客请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

在 [C# dotnet 使用 OpenXml 解析 PPT 文件](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-PPT-%E6%96%87%E4%BB%B6.html) 这篇博客中没有详细告诉大家页面顺序的问题，但是按照 [C# dotnet 使用 OpenXml 解析 PPT 文件](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-PPT-%E6%96%87%E4%BB%B6.html) 这篇博客的写法就是能拿到对的页面顺序

在 ECMA 376 标准中说明，在 Presentation.xml 文档将会记录页面的顺序和页面的 Id 值，也就是 Slide Id 值，代码大概如下

```xml
  <p:sldIdLst>
    <p:sldId id="277" r:id="rId2" />
  </p:sldIdLst>
```

这里的 `p:sldIdLst` 将会存放在 PPT 画布里面多个页面之间的顺序，上面代码中页面的 Slide Id 是 `id` 这个属性，也就是当前的文档只有一个页面，这个页面的 Slide Id 是 `277` 的值。而后面的 `r:id="rId2"` 这个指的是文档压缩包里面的资源路径，可以通过这个属性找到对应的页面数据，请看下面代码

```csharp
                var slideIdList = presentation.SlideIdList;

                foreach (var slideId in slideIdList.ChildElements.OfType<SlideId>())
                {
                    // 获取页面内容
                    SlidePart slidePart = (SlidePart) presentationPart.GetPartById(slideId.RelationshipId);
                    // 忽略代码
                }
```

上面代码的 `slideId.RelationshipId` 就是对应 `r:id` 属性。这个属性是通用的属性，详细请看 [Office 文档解析 文档格式和协议](https://blog.lindexi.com/post/Office-%E6%96%87%E6%A1%A3%E8%A7%A3%E6%9E%90-%E6%96%87%E6%A1%A3%E6%A0%BC%E5%BC%8F%E5%92%8C%E5%8D%8F%E8%AE%AE.html )

而 `id="277"` 的 `id` 需要使用 `slideId.Id` 读取，请看下面代码

```csharp
public void Foo(FIleInfo file)
{
  using var document = PresentationDocument.Open(file.OpenRead(), isEditable: false);
  var openXmlPresentation = document.PresentationPart.Presentation;
  var slideIdList = openXmlPresentation.SlideIdList;
  foreach (var slideId in slideIdList.ChildElements.OfType<SlideId>())
  {
      var id = slideId.Id;
  }
}
```

如何了解自己读取到的值是否是对的？试试使用 COM 的方式，或者创建一个 VSTO 插件，试试使用下面代码获取

```csharp
            var application = new Application();
            var presentation = application.Presentations.Open(file, MsoTriState.msoTrue,
                MsoTriState.msoFalse,
                MsoTriState.msoFalse);

            foreach (Microsoft.Office.Interop.PowerPoint.Slide presentationSlide in presentation.Slides)
            {
                var slideId = presentationSlide.SlideID;
            }
```

这部分细节在 ECMA 376 文档提到的不多，请看 19.2.1.33 sldId (Slide ID)  和 19.2.1.34 sldIdLst (List of Slide IDs)  这两章


我写了很多 Office 解析相关的博客，请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
