
# dotnet OpenXML SDK 文本占位符解析

在使用 OpenXML SDK 解析 PPT 文档的文本占位符的时候，需要对 PPT 的格式有一定的了解，尽管整个 OpenXML SDK 包括文档等都很详细。但是有一些细节文档上虽然有写，但是没有强调一下，就被我忽略了

<!--more-->


<!-- CreateTime:5/9/2020 10:43:59 AM -->

<!-- 发布 -->

什么是文本占位符，其实这是在 PPT 添加的概念，在 PPT 里面用户可以编辑模版文件，在这里定义某个占位符文本的样式和坐标等

如何制作占位符请看 [PPT占位符，居然这么好用！ - 知乎](https://zhuanlan.zhihu.com/p/37086389 )

想要解析占位符还需要先学会如何使用占位符才好理解占位符是如何做的

在 OpenXML 里面文本是形状，也就是 `DocumentFormat.OpenXml.Presentation.Shape` 元素，可以使用下面代码获取页面的形状

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

                    foreach (var shape in
                        slidePart.Slide
                            .Descendants<DocumentFormat.OpenXml.Presentation.Shape>())
                    {
                       
                    }
                }
            }
```

在 PPT 里面是使用 `p:ph` 判断一个形状是占位符，下面是一个占位符的形状

```xml
<p:sp>
    <p:nvsppr>
        <p:cnvpr id="2" name="标题 1">
        </p:cnvpr>
        <p:cnvsppr>
            <a:splocks nogrp="1">
            </a:splocks>
        </p:cnvsppr>
        <p:nvpr>
            <p:ph type="ctrTitle">
            </p:ph>
        </p:nvpr>
    </p:nvsppr>
    <p:sppr>
    </p:sppr>
    <p:txbody>
        <a:bodypr>
        </a:bodypr>
        <a:p>
            <a:r>
                <a:rpr altlang="en-US" lang="zh-CN">
                </a:rpr>
                <a:t>
                    PPT 解析
                </a:t>
            </a:r>
            <a:endpararpr altlang="en-US" lang="zh-CN">
            </a:endpararpr>
        </a:p>
    </p:txbody>
</p:sp>
```

在 slide.xml 里面的元素，通过设置 nvsppr->nvpr->ph 设置这个元素使用占位符，需要继承模版的占位符样式和坐标等值

从 Shape 里面拿到占位符可以使用下面代码

```csharp
// <p:nvSpPr>占位符的样式
NonVisualShapeProperties nonVisualShapeProperties = shape?.NonVisualShapeProperties;
// cNvSpPr
var nonVisualShapeDrawingProperties = nonVisualShapeProperties?.NonVisualShapeDrawingProperties;
// nvpr
var applicationNonVisualDrawingProperties = nonVisualShapeProperties?.ApplicationNonVisualDrawingProperties;

var placeholderShape = applicationNonVisualDrawingProperties?.PlaceholderShape;
```

可以在 placeholderShape 里面找到两个主要的属性，一个是 Index 一个是 Type 属性，这两个属性是什么意思？从属性的注释可以看到写的很复杂，大概的做法就是占位符需要去找到模版里面相同的 Index 或相同的 Type 的占位符元素，获取这个元素的样式和坐标等

如果有仔细阅读上面文档就可以知道，如果用户在模版里面定义了占位符，那么仅仅表示页面里面的对应元素的默认样式，而元素本文可以覆盖此样式。也就是元素的最终样式是先尝试获取元素本文的样式，如果元素本文获取不到样式，那么尝试运行占位符元素，如果可以找到占位符元素，那么尝试获取占位符元素的对应样式

那么如何通过 placeholderShape 找到对应的放在模版里面的占位符元素？是否小伙伴还记得 Slide Layout 和 Slide Master 的概念，如果不知道的话，请复习一下 PPT 是如何制作的课程，这两个概念有点绕，需要小伙伴学会制作 PPT 才比较好说

获取 SlideLayout 和 SlideMaster 可以使用下面代码

```csharp
var layout = slidePart.SlideLayoutPart.SlideLayout;
var master = slidePart.SlideLayoutPart.SlideMasterPart.SlideMaster;
```

先从 layout 里面尝试找到有没有对应的占位符元素，如果没有找到再从 master 里面找

寻找方法是从 CommonSlideData 的 ShapeTree 寻找是否有对应的元素，那么什么是对应的元素，如果页面元素设置了 Type 那么要求 ShapeTree 的元素的占位符属性有完全相同的 Type 属性，如果页面元素设置了 Index 那么要求 ShapeTree 的有相同的 ShapeTree 属性。如果页面元素的 Type 是空，那么就不对 ShapeTree 的属性有要求，如果 Index 是空，那么对 ShapeTree 的属性也没有要求

```csharp
        private static Shape GetPlaceholderShape(PlaceholderShape placeholder, ShapeTree tree)
        {
            return tree.Elements<Shape>().FirstOrDefault(shape =>
            {
                var placeholderShape = shape?.NonVisualShapeProperties?.ApplicationNonVisualDrawingProperties
                    ?.PlaceholderShape;
                return placeholderShape != null && Equals(placeholder, placeholderShape);
            });
        }

        /// <summary>
        /// 比较<see cref="PlaceholderShape"/>的type和id是否相同
        /// <para></para>
        /// 如果 1 的 Type 或 Index 是空，那么跳过这个属性的判断
        /// <para></para>
        /// 如果这个属性不是空，那么一定要求 2 存在这个属性
        /// </summary>
        /// 这个规则通过 文本占位符没有type和id的值，获取第一个占位符作为坐标 和 WPS 对比测试拿到
        /// 测试课件：文本占位符没有type和id的值.pptx
        /// <param name="placeholder1"></param>
        /// <param name="placeholder2"></param>
        /// <returns></returns>
        private static bool Equals(PlaceholderShape placeholder1, PlaceholderShape placeholder2)
        {
            // 如果 placeholder1.Type 存在值，要求 2 一定存在值
            if (placeholder1.Type != null && 
                placeholder1.Type.Value != placeholder2.Type?.Value)
            {
                return false;
            }

            if (placeholder1.Index != null && placeholder1.Index.Value !=
                placeholder2.Index?.Value)
            {
                return false;
            }

            return true;
        }
```

获取的方法如下

```csharp
  layoutPlaceholder = GetPlaceholderShape(placeholder, layout?.CommonSlideData?.ShapeTree);
  masterPlaceholder = GetPlaceholderShape(placeholder, master?.CommonSlideData?.ShapeTree);
```

此时的样式获取顺序就是先从元素获取，如果元素获取不到，就从 layoutPlaceholder 获取，如果获取不到从 masterPlaceholder 获取

注释里面的 文本占位符没有type和id的值.pptx 我就不放出来了，有需要的小伙伴发邮件给我

更多的 OpenXML 相关博客，还请自行百度 `OpenXML 林德熙` 就能找到我的博客了





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。