# C# dotnet 使用 OpenXml 解析 PPT 文件

在 2013 微软开源了 OpenXml 解析库，在微软的 PPTX 文档，使用的文档格式就是国际规范的 OpenXml 格式。这个格式有很多版本，详细请看百度。因为演示文稿使用的是 OpenXml 在 .NET 开发可以非常简单将 PowerPointer 文档进行解析，大概只需要两句话

<!--more-->
<!-- CreateTime:2020/2/29 10:27:27 -->

<!-- csdn -->

解析 PPT 文件不等于显示 PPT 文件，只是可以拿到 PPT 里面的数据

第一步是通过 NuGet 安装 [Openxml](https://www.nuget.org/packages/DocumentFormat.OpenXml) 库，这个库支持跨平台，因为只是解析数据

第二步就是传入 PPT 文件解析

```csharp
            using (var presentationDocument = DocumentFormat.OpenXml.Packaging.PresentationDocument.Open("测试.pptx", false))
```

这样就完成了 PPT 文件的解析，在调试添加断点，可以在局部变量看到 presentationDocument 的内容

这里面的内容就是整个 PPT 的数据，至于这些数据的含义是什么，就需要额外阅读一下文档

下面是一个简单的例子，获取 PPT 文件里面每一页的所有文本

```csharp
            using (var presentationDocument = DocumentFormat.OpenXml.Packaging.PresentationDocument.Open("测试.pptx", false))
            {
                var presentationPart = presentationDocument.PresentationPart;
                var presentation = presentationPart.Presentation;

                // 先获取页面
                var slideIdList = presentation.SlideIdList;

                foreach (var slideId in slideIdList.ChildElements.OfType<SlideId>())
                {
                    // 获取页面内容
                    SlidePart slidePart = (SlidePart) presentationPart.GetPartById(slideId.RelationshipId);

                    var slide = slidePart.Slide;

                    foreach (var paragraph in
                        slidePart.Slide
                            .Descendants<DocumentFormat.OpenXml.Drawing.Paragraph>())
                    {
                        // 获取段落
                        // 在 PPT 文本是放在形状里面
                        foreach (var text in
                            paragraph.Descendants<DocumentFormat.OpenXml.Drawing.Text>())
                        {
                            // 获取段落文本，这样不会添加文本格式
                            Debug.WriteLine(text.Text);
                        }
                    }
                }
            }
```

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/3bb1678686dbd12c4b2d911d3d3bd42ec30d8987/WhocohefurWallqemwaychurgu) 欢迎小伙伴访问

[OfficeDev/Open-XML-SDK: Open XML SDK by Microsoft](https://github.com/OfficeDev/Open-XML-SDK )

[Openxml学习 - 标签 - FrankZC - 博客园](https://www.cnblogs.com/FourLeafCloverZc/tag/Openxml%E5%AD%A6%E4%B9%A0/ )

[Open Xml SDK 引文](https://www.cnblogs.com/pengzhen/p/3811834.html )

官方文档 [欢迎使用 Open XML SDK 2.5 for Office](https://docs.microsoft.com/zh-cn/office/open-xml/open-xml-sdk )

其他语言的解析

[scanny/python-pptx: Create Open XML PowerPoint documents in Python](https://github.com/scanny/python-pptx )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
