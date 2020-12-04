
# dotnet OpenXML SDK 文本占位符解析

在使用 OpenXML SDK 解析 PPT 文档的文本占位符的时候，需要对 PPT 的格式有一定的了解，尽管整个 OpenXML SDK 包括文档等都很详细。但是有一些细节文档上虽然有写，但是没有强调一下，就被我忽略了

<!--more-->


<!-- CreateTime:5/9/2020 10:43:59 AM -->



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
        <p:cNvPr id="2" name="标题 1">
        </p:cNvPr>
        <p:cNvSpPr>
            <a:splocks nogrp="1">
            </a:splocks>
        </p:cNvSpPr>
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

> 下面这句话是错的
> 先从 layout 里面尝试找到有没有对应的占位符元素，如果没有找到再从 master 里面找

无论是 SlideLayout 还是 SlideMaster 都在 CommonSlideData 的 ShapeTree 存放占位符元素。因此寻找占位符方法是从 CommonSlideData 的 ShapeTree 寻找是否有对应的元素，那么什么是对应的元素，如果页面元素设置了 Type 那么要求 ShapeTree 的元素的占位符属性有完全相同的 Type 属性，如果页面元素设置了 Index 那么要求 ShapeTree 的有相同的 ShapeTree 属性。如果页面元素的 Type 是空，那么就不对 ShapeTree 的属性有要求，如果 Index 是空，那么对 ShapeTree 的属性也没有要求

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

> 下面这句话是不对的
> 此时的样式获取顺序就是先从元素获取，如果元素获取不到，就从 layoutPlaceholder 获取，如果获取不到从 masterPlaceholder 获取

注释里面的 文本占位符没有type和id的值.pptx 我就不放出来了，有需要的小伙伴发邮件给我

更多的 OpenXML 相关博客，还请自行百度 `OpenXML 林德熙` 就能找到我的博客了

更正一下

小伙伴可以看到我标记了文章一些说法是不对的。原因是 ECMA 376 文档里面其实只包含了 Placeholder 的定义，而没有包含他的实现方式。整个 [ECMA](http://www.ecma-international.org/publications/standards/Ecma-376.htm) 关于 Placeholder 仅有 274 个引用。因此我上面这里的说法完全只是因为没有实践而依靠不靠谱的博客找到的方法

以下为我通过 Office 2013 和 Office 2016 和 WPS 11.3.0.8742 版本测试拿到的规则

- 占位符元素需要同时在 SlideLayout 和 SlideMaster 里面查找
- 占位符元素的属性优先级是 Slide 里元素本身，然后是 SlideLayout 占位符元素最后是 SlideMaster 占位符元素
  + 假定尝试获取元素的平移属性，此时在元素本身没有找到，就需要从 SlideLayout 占位符元素（如果存在）里尝试获取平移属性
  + 假定从 SlideLayout 占位符元素获取不到平移属性，那么尝试从 SlideMaster 占位符元素获取平移属性
- 占位符元素如果设置了 Id 的值，那么标准文档里面这个 Id 在 SlideLayout 和 SlideMaster 仅能找到一个占位符元素，不会存在两个
  + 对 WPS 非标准文档，可能存在两个相同 Id 元素，此时使用 xml 的第一个元素
- 占位符元素如果设置了 Id 的值，在 SlideMaster 里面没有找到对应的 Id 的占位符元素，那么尝试通过占位符元素的 Type 找到对应的 SlideMaster 的元素
- 如果占位符元素没有设置 Type 的值，那么默认值是 Body 的值
- 如果在 SlideMaster 里面存在多个 Body 元素，那么标准文档里面将会设置每个 Body 元素都有 Id 的值
 + 对 WPS 非标准文档，如果定义多个 Body 元素，且没有给每个 Body 元素设置 Id 的值，那么使用 xml 的第一个 Body 元素
 + 对 WPS 非标准文档，如果定义多个 Body 元素，其中有一个 Body 元素没有 Id 的值，且使用 Id 查找不到对应占位符元素，那么使用第一个没有 Id 的 Body 元素

大概逻辑如下，下面代码仅使用 SlideMaster 的占位符元素，原因是 SlideLayout 没有遇到此非标准文档，而我也不想去改文档代码测试

```csharp
        /// <summary>
        /// 对 SlideMaster 的获取占位符的规则是假设 PlaceholderShape 存在 Id 的值，那么在 SlideMaster 所有元素尝试找到对应的 Id 的值的元素，如果能找到那么这个元素就是占位符元素。如果不存在 Id 或找不到对应的元素，那么进行 Type 的查找，如果传入的 PlaceholderType 没有设置值，那么将使用默认的 PlaceholderValues.Body 的值
        /// </summary>
        /// <returns></returns>
        private static Shape GetMasterPlaceholderByPlaceHolderType(PlaceholderShape placeholder, SlideMaster master)
        {
            EnumValue<PlaceholderValues> placeholderType = placeholder.Type;
            const PlaceholderValues defaultPlaceholderValue = PlaceholderValues.Body;

            var type = placeholderType?.Value ?? defaultPlaceholderValue;
            var id = placeholder.Index?.Value;

            var elementList = master?.CommonSlideData?.ShapeTree?.Elements<Shape>();
            if (elementList == null)
            {
                return null;
            }

            Shape typeMatchShape = null;

            foreach (var shape in elementList)
            {
                var placeholderShape = shape?.NonVisualShapeProperties?.ApplicationNonVisualDrawingProperties
                    ?.PlaceholderShape;

                if (placeholderShape == null)
                {
                    continue;
                }

                if (id != null)
                {
                    // 优先找到 id 相同的占位符
                    // 如果 id 相同的找不到，那么找 Type 相同的
                    if (placeholderShape?.Index?.Value == id.Value)
                    {
                        return shape;
                    }
                }

                // 以下逻辑不全对，注意在有两个Body元素，此时 id != null 且这两个 Body 元素
                // 第一个元素存在 Id 且和 placeholder.Index 不相等
                // 第二个元素不存在 Id 的值 
                // 那么此时应该选用第二个元素，不应该选择第一个元素
                // 但是下面代码选用的是第一个元素
                if (placeholderShape.Type?.Value == type)
                {
                    // 基本只有一个 Type 相等，如果有多个，那么这个课件不是标准的
                    Debug.Assert(typeMatchShape == null);
                    typeMatchShape = shape;
                }
            }

            return typeMatchShape;
        }
```

上面代码的逻辑不全对，我写在注释里面

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。