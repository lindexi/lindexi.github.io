# C# dotnet 使用 OpenXml 解析 PPT 里面的视频

本文告诉大家如何从 PPTX 文件里面解析出视频

<!--more-->
<!-- CreateTime:2020/3/13 19:09:15 -->



我期望看到本文的小伙伴是了解 OpenXML 的，如果想要解析 Office 的文档，我推荐使用使用 OpenXML SDK 这个开源的库，更多入门级博客请看 [C# dotnet 使用 OpenXml 解析 PPT 文件](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-PPT-%E6%96%87%E4%BB%B6.html)

我做了一个简单的 PPT 文件，这个文件里面只有一页，这一页上面有一个视频。做这个文件的作用是方便调试，本文将从这个文件里面拿到视频

```csharp
            using (var presentationDocument =
                DocumentFormat.OpenXml.Packaging.PresentationDocument.Open(@"小视频.pptx", false))
            {
                var presentationPart = presentationDocument.PresentationPart;
                var slidePart = presentationPart.SlideParts.FirstOrDefault();

                // 忽略代码
            }
```

上面代码是打开解析文件，我拿到第一页，而获取页面的元素需要了解一点是 PPT 将所有元素存放 ShapeTree 里面。相当于所有的元素都放在形状树里面。在 PPT 里面是不存在视频元素的，没有一个叫做视频的元素，在 PPT 里面用 Picture 存放视频。也就是说视频也是一张图片，图片也是一张图片。不过视频的图片采用的是视频的缩略图。在解析的时候，需要先尝试当成视频解析，视频解析失败了，再当成图片解析

在 OpenXML 的存储的视频大概如下

```xml
<p:pic>
    <p:nvpicpr>
        <p:cnvpr id="4" name="视频">
            <a:hlinkclick action="ppaction://media" r:id="">
            </a:hlinkclick>
        </p:cnvpr>
        <p:cnvpicpr>
            <a:piclocks nochangeaspect="1">
            </a:piclocks>
        </p:cnvpicpr>
        <p:nvpr>
            <a:videofile r:link="rId2">
            </a:videofile>
            <p:extlst>
                <p:ext uri="{DAA4B4D4-6D71-4841-9C94-3DE7FCFB9230}">
                    <p14:media r:embed="rId1" xmlns:p14="http://schemas.microsoft.com/office/powerpoint/2010/main">
                    </p14:media>
                </p:ext>
            </p:extlst>
        </p:nvpr>
    </p:nvpicpr>
</p:pic>
```

如上面代码，这就是视频元素其实也就是 Picture 元素，可以在 `picture.NonVisualPictureProperties.ApplicationNonVisualDrawingProperties` 属性里面找到 VideoFromFile 对象，通过此对象获取到视频内容，从而解析视频

```csharp
    var picture = slidePart.Slide.CommonSlideData.ShapeTree.OfType<Picture>().FirstOrDefault();
    var videoFromFile = picture.NonVisualPictureProperties
        .ApplicationNonVisualDrawingProperties
        .GetFirstChild<VideoFromFile>();
```

如上面的 OpenXML 代码，可以看到视频是通过 `r:link` 关联对应的资源。获取视频内容的方法用的是 `r:link` 拿到对应的资源。在 OpenXmlSDK 里面，获取 Relationship 有两个不同的方式，用 GetPartById 获取 ChildrenRelationshipParts 的资源，用 GetReferenceRelationship 拿到 ReferenceRelationshipList 的资源。由于在 PPT 里面的视频放在 ReferenceRelationshipList 里，可以使用使用下面代码拿到

```csharp
           var openXmlPart = (DataPartReferenceRelationship) slidePart.GetReferenceRelationship(videoFromFile.Link.Value);
```

通过 GetStream 方法可以拿到压缩包里面的文件，这里不需要解压缩，在底层已自动解压缩内容，新建文件写入就可以

```csharp
                var stream = openXmlPart.DataPart.GetStream();
                var file = System.IO.Path.Combine(System.IO.Path.GetTempPath(), "林德熙是逗比.mp4");
                File.WriteAllBytes(file, ReadAllBytes(stream));
```

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8551ad78455f7e56e2f1cafa66d6ae62d7a94995/GairhajelkewaiHeyerjeaginu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/8551ad78455f7e56e2f1cafa66d6ae62d7a94995/GairhajelkewaiHeyerjeaginu) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 8551ad78455f7e56e2f1cafa66d6ae62d7a94995
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 GairhajelkewaiHeyerjeaginu 文件夹

以上是大部分的 PPT 存放视频的存储方式，通过以上方法可以获取大多数课件里的视频

然而还有一部分比较特殊的，是需要通过 `DAA4B4D4-6D71-4841-9C94-3DE7FCFB9230` 扩展获取。在上面的 OpenXML 存储代码里面，在 `a:videofile` 下面还有一段 `p:extlst` 的代码。这里的 `p:extlst` 就是 `DocumentFormat.OpenXml.Presentation.ApplicationNonVisualDrawingPropertiesExtensionList` 类型，表示的是存储扩展类型，存储扩展里面，可以通过 Uri 指定此扩展的 Guid 从而指定这是一个什么扩展

```xml
        <p:nvpr>
            <a:videofile r:link="rId2">
            </a:videofile>
            <p:extlst>
                <p:ext uri="{DAA4B4D4-6D71-4841-9C94-3DE7FCFB9230}">
                    <p14:media r:embed="rId1" xmlns:p14="http://schemas.microsoft.com/office/powerpoint/2010/main">
                    </p14:media>
                </p:ext>
            </p:extlst>
        </p:nvpr>
```

以上的给视频元素使用的 `DAA4B4D4-6D71-4841-9C94-3DE7FCFB9230` 是一段特殊的扩展，叫做 `media extension` 媒体扩展，可以通过 ECMA 376 的第 2.3.1.18 章了解相关的信息

在 `DAA4B4D4-6D71-4841-9C94-3DE7FCFB9230` 扩展里面存放的多媒体，可以附加储存以下功能

- 进行软裁剪，设置视频的循环播放周期，设置起始点和结束点，让视频在起始点和结束点中间播放
- 提供设置视频的淡入功能
- 允许在视频中设置书签，以便快速跳转到特定位置

本文这里只告诉大家如何获取到视频文件

对于某些特殊的文件来说，在 `<a:videoFile r:link="rId1" />` 记录的内容也许只是一个空对象而已。如对应的 `rId1` 的内容如下

```xml
  <Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/video" Target="NULL" TargetMode="External" />
```

可以看到这里只是连接到一个 `Target="NULL"` 的外部内容，也就是一个空对象。如此是不能通过上文的方式获取到视频文件的，需要通过以下的方式获取

先获取到扩展列表 `DocumentFormat.OpenXml.Presentation.ApplicationNonVisualDrawingPropertiesExtensionList` 对象，接着根据 `DAA4B4D4-6D71-4841-9C94-3DE7FCFB9230` 获取到具体的扩展

```csharp
            // 这里的 ApplicationNonVisualDrawingProperties 包含以下内容
            /*
                      <p:nvPr>
                        <a:videoFile r:link="rId1" />
                        <p:extLst>
                          <p:ext uri="{DAA4B4D4-6D71-4841-9C94-3DE7FCFB9230}">
                            <p14:media xmlns:p14="http://schemas.microsoft.com/office/powerpoint/2010/main" r:embed="rId2">
                              <p14:trim st="109898" end="23655" />
                            </p14:media>
                          </p:ext>
                        </p:extLst>
                      </p:nvPr>
             */
            // 获取 <p:extLst> 的内容
            var extensionList = applicationNonVisualDrawingProperties!.GetFirstChild<DocumentFormat.OpenXml.Presentation.ApplicationNonVisualDrawingPropertiesExtensionList>();

            // 获取特殊的多媒体扩展
            /*
            Extension DAA4B4D4-6D71-4841-9C94-3DE7FCFB9230 is described as a media extension. It appears to allow:

                - “cropping” the video period (set start and stop time markers)
                - provide for “fade-in”
                - allow for setting bookmarks in the video for fast jumps to a particular location
             */
            var mediaExtension = extensionList?.Elements<DocumentFormat.OpenXml.Presentation.ApplicationNonVisualDrawingPropertiesExtension>().FirstOrDefault(extension => extension.Uri?.Value == "{DAA4B4D4-6D71-4841-9C94-3DE7FCFB9230}");
```

通过媒体扩展获取到 `DocumentFormat.OpenXml.Office2010.PowerPoint.Media` 也就是 `p14:media` 的内容

```csharp
            if (mediaExtension != null)
            {
                // p14:media
                var media = mediaExtension.GetFirstChild<DocumentFormat.OpenXml.Office2010.PowerPoint.Media>();
            }
```

接着如上文的方法，通过 `media.Embed` 获取到对应的资源，代码如下

```csharp
                var currentPart = slidePart;

                var media = mediaExtension.GetFirstChild<DocumentFormat.OpenXml.Office2010.PowerPoint.Media>();
                var embedValue = media?.Embed?.Value;
                if (embedValue != null)
                {
                    var videoPart = currentPart.GetReferenceRelationship(embedValue);

                    return videoPart as DataPartReferenceRelationship;
                }
```

获取到 `videoPart` 即可使用上面的代码的方式保存为本地文件

参阅 [C# Net 使用 openxml 提取ppt中的音频、视频、图片、文本 - 爱恋的红尘 - 博客园](https://www.cnblogs.com/ping9719/p/13497923.html )

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
