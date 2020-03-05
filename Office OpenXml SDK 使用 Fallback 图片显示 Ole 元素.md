# Office OpenXml SDK 使用 Fallback 图片显示 Ole 元素

我在写一个 WinForms 程序用来读取 Word 里面的图片显示，在解析 Word 等 Office 文档，会看到一些 ole object 元素，而有些 ole object 会有 Fallback 图片，用这些备用的图片可以显示 ole 元素

<!--more-->
<!-- CreateTime:2020/3/3 8:23:31 -->

<!-- 发布 -->

其实有很多 Office 插件公司在开发，而特殊的元素如何在其他版本打开？或者我用插件做了一个复杂的元素，在没有插件的设备如何让用户看到？在 Office 的一个做法是通过 Fallback 元素，在里面放一张图片

因为我的 Word 文档写了很多逗比的话，就不开放给大家。除了 Word 在 PPT 解析上也差不多，解析 PPT 里面的 Ole 元素，使用 Fallback 元素显示图片是本文的例子。这份文档也不能给大家，我不觉得你没事干会看本文，应该是你遇到了 Office 解析 ole 元素如何显示或 oleobj 如何转换等问题会看本文 ，也就是你其实有一份 Office 文档了

我将这个文档放在 "F:\林德熙是逗比" 文件夹，也就是你拿到我的[代码](https://github.com/lindexi/lindexi_gd/tree/d182ca9f0cece56d32a801923a1fdffa64f95dfd/NallwerewawchailawileeForeehakel)也许需要更改一下代码里面的路径，才能跑起来

先安装 DocumentFormat.OpenXml 库，这是一个完全开源的官方的全平台的库

```csharp
  <ItemGroup>
    <PackageReference Include="DocumentFormat.OpenXml" Version="2.10.1" />
  </ItemGroup>
```

是不是觉得我上面代码安装库很奇特，其实这是 SDK Style 格式的 csporj 的写法，可以瞬间安装完成一个 NuGet 库。如何使用这个格式请看 [从以前的项目格式迁移到 VS2017 新项目格式](https://blog.lindexi.com/post/%E4%BB%8E%E4%BB%A5%E5%89%8D%E7%9A%84%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E8%BF%81%E7%A7%BB%E5%88%B0-VS2017-%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F.html)

通过下面代码可以打开解析 Office 文件，本文打开的是一个 PPT 文件

```csharp
            using (var doc = PresentationDocument.Open(pptxFilePath, false))
```

我推荐这部分可以放在后台代码，因为 PresentationDocument.Open 需要做的内容会比较多

上面如何打开 PPT 请看 [C# dotnet 使用 OpenXml 解析 PPT 文件](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-PPT-%E6%96%87%E4%BB%B6.html)

我假定只有一个页面，因为我传入的PPT文件就只有一个页面，这个需要根据你的实际代码更改

```csharp
                // 我假定你只有一个页面
                var slidePart = doc.PresentationPart.SlideParts.First();
```

元素是放在页面里面，也就是需要拿到页面

```csharp
                // 拿到页面
                var slide = slidePart.Slide;
```

很多 ole 元素都放在 `p:graphicframe` 里面，先尝试遍历所有 GraphicFrame 请看代码

```csharp
                // 找到所有也许是 ole 的元素
                foreach (var frame in slide.CommonSlideData.ShapeTree
                    .OfType<DocumentFormat.OpenXml.Presentation.GraphicFrame>())
```

在原文的写法是

```xml
<p:graphicframe>
    <p:nvgraphicframepr>
        <p:cnvpr id="8" name="Object 25">
        </p:cnvpr>
        <p:cnvgraphicframepr>
            <a:graphicframelocks nochangeaspect="1">
            </a:graphicframelocks>
        </p:cnvgraphicframepr>
        <p:nvpr>
            <p:extlst>
            </p:extlst>
        </p:nvpr>
    </p:nvgraphicframepr>
    <a:graphic>
        <a:graphicdata uri="http://schemas.openxmlformats.org/presentationml/2006/ole">
            <mc:alternatecontent xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006">
                <mc:choice requires="v" xmlns:v="urn:schemas-microsoft-com:vml">
                    <p:oleobj imgh="2535890" imgw="5253016" name="文档" progid="Word.Document.12" r:id="rId3" spid="_x0000_s1026">
                        <p:embed>
                        </p:embed>
                    </p:oleobj>
                </mc:choice>
                忽略一些代码
            </mc:alternatecontent>
        </a:graphicdata>
    </a:graphic>
</p:graphicframe>
```

也就是 ole 元素 `p:oleobj` 放在 graphic 里面，不过在 OpenXML SDK 可以使用 Linq 的方式快速读取到对应的值

```csharp
var oleElement = frame.Descendants<DocumentFormat.OpenXml.Presentation.OleObject>()
    .FirstOrDefault();
if (oleElement != null)
{
}
```

然后尝试读取 oleElement 的 Fallback 是否有图片

不是所有的 ole element 都有备用的图，需要看你的文档里面是否有 `mc:fallback` 元素，同时这个元素是 `p:pic` 图片元素

在Office的图片填充用的是 [p:blipFill](https://docs.microsoft.com/en-us/dotnet/api/documentformat.openxml.presentation.blipfill?view=openxml-2.8.1 ) 而这个元素需要用到依赖的元素

简单的图片是

```
<p:pic>  
  …  
  <p:blipFill>  
    <a:blip r:embed="rId2"/>  
    <a:stretch>  
      <a:fillRect/>  
    </a:stretch>  
  </p:blipFill>  
  …  
</p:pic>
```

核心在于 `r:embed` 的值，这个值可以从 `xml.rel` 里面读取，但是这里的读取逻辑很复杂。不过 OpenXML SDK 已经封装了

那么如何从拿到 OleObject 返回备用图片，先拿到对应的页面，所有资源放在页面的 SlidePart 元素

```csharp
        private static (bool isSuccess, FileInfo file) TryGetFallbackImage(OleObject oleElement)
        {
            var slide = oleElement.Ancestors<Slide>().First();

            var slidePart = slide.SlidePart;

            // 忽略代码
        }
```

在 OpenXML 用的类是继承 XmlDocument 也就是可以通过 oleElement 向上找到 `p:graphicframe` 元素

```csharp
            var frame = oleElement.Ancestors<GraphicFrame>().First();

            var frameGraphic = frame.Graphic.GraphicData;
```

先补充 `mc:fallback` 的文档

```xml
<a:graphicdata uri="http://schemas.openxmlformats.org/presentationml/2006/ole">
    <mc:alternatecontent xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006">
        <mc:choice requires="v" xmlns:v="urn:schemas-microsoft-com:vml">
            <p:oleobj imgh="2535890" imgw="5253016" name="文档" progid="Word.Document.12" r:id="rId3" spid="_x0000_s1026">
                <p:embed>
                </p:embed>
            </p:oleobj>
        </mc:choice>
        <mc:fallback>
            <p:oleobj imgh="2535890" imgw="5253016" name="文档" progid="Word.Document.12" r:id="rId3">
                <p:embed>
                </p:embed>
                <p:pic>
                    <p:nvpicpr>
                        <p:cnvpr id="0" name="">
                        </p:cnvpr>
                        <p:cnvpicpr>
                            <a:piclocks nochangearrowheads="1" nochangeaspect="1">
                            </a:piclocks>
                        </p:cnvpicpr>
                        <p:nvpr>
                        </p:nvpr>
                    </p:nvpicpr>
                    <p:blipfill>
                        <a:blip r:embed="rId4">
                            <a:extlst>
                                <a:ext uri="{28A0092B-C50C-407E-A947-70E740481C1C}">
                                    <a14:uselocaldpi val="0" xmlns:a14="http://schemas.microsoft.com/office/drawing/2010/main">
                                    </a14:uselocaldpi>
                                </a:ext>
                            </a:extlst>
                        </a:blip>
                        <a:srcrect>
                        </a:srcrect>
                        <a:stretch>
                            <a:fillrect>
                            </a:fillrect>
                        </a:stretch>
                    </p:blipfill>
                    </p:sppr>
                </p:pic>
            </p:oleobj>
        </mc:fallback>
    </mc:alternatecontent>
</a:graphicdata>
```

从上面文档代码可以看到 `mc:fallback` 可以通过 `frameGraphic.Descendants<DocumentFormat.OpenXml.AlternateContentFallback>().FirstOrDefault()` 拿到

而对应的图片可以使用下面代码拿到

```csharp
            var fallback = frameGraphic.Descendants<DocumentFormat.OpenXml.AlternateContentFallback>().FirstOrDefault();
            if (fallback == null)
            {
                return (false, null);
            }

            var picture = fallback.Descendants<DocumentFormat.OpenXml.Presentation.Picture>().FirstOrDefault();

            if (picture == null)
            {
                return (false, null);
            }
```

重要的代码是如何拿到 picture 的 `a:blip r:embed="rId4"` 的 rId4 的图片

在 OpenXML SDK 定义好了 BlipFill 可以通过下面代码拿到 rId 的值

```csharp
            var embed = picture.BlipFill.Blip.Embed.Value;
```

而拿到 embed 就可以拿到对应的 Stream 可以写入文件

```csharp
            var part = slidePart.GetPartById(embed);

            if (part is ImagePart imagePart)
            {
                if (imagePart.ContentType == "image/x-wmf" || imagePart.ContentType == "image/x-emf")
                {
                    var stream = part.GetStream(FileMode.Open, FileAccess.Read);
                    var fileName = Path.GetFileName(imagePart.Uri.OriginalString);
                    var file = Path.Combine(@"F:\林德熙是逗比", fileName);
                    File.WriteAllBytes(file, ReadAllBytes(stream));

                    return (true, new FileInfo(file));
                }
            }
```

上面代码写入文件是 "F:\林德熙是逗比" 小伙伴需要按照你的需求更改，上面代码也没有释放资源

这里的 ReadAllBytes 通过将 Stream 转 byte[] 方法

```csharp
        private static byte[] ReadAllBytes(Stream stream)
        {
            using (var memoryStream = new MemoryStream())
            {
                stream.CopyTo(memoryStream);
                return memoryStream.ToArray();
            }
        }
```

在 OpenbXML SDK 可以方便拿到资源，通过 `var part = slidePart.GetPartById(embed)` 方法，此时返回的是 part 可以用 GetStream 返回压缩包里面的资源

其实在 WinForms 可以通过更简单的代码创建图片

```csharp
System.Drawing.Image img = System.Drawing.Image.FromStream(imagePart.GetStream());
```

这样就能完成在 Office 文件解析 ole 元素，但是只要 ole 元素没有写 Fallback 本文方法也没有用

如果我只有 ole 元素，我能否显示，有大神写了 [The DotNet Heaven: Read OLE Object type image field in C#.net](http://thedotnetheaven.blogspot.com/2010/03/read-ole-object-type-image-field-in.html )

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/d182ca9f0cece56d32a801923a1fdffa64f95dfd/NallwerewawchailawileeForeehakel) 欢迎小伙伴访问，如果无法下载源代码，请到 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/d182ca9f0cece56d32a801923a1fdffa64f95dfd/NallwerewawchailawileeForeehakel) 下载

[How to parse embedded file(OLE obejct) in pptx/docx · Issue #644 · OfficeDev/Open-XML-SDK](https://github.com/OfficeDev/Open-XML-SDK/issues/644 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
