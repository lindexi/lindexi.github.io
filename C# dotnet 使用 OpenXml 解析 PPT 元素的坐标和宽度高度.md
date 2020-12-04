# C# dotnet 使用 OpenXml 解析 PPT 元素的坐标和宽度高度

在阅读本文之前，我期望你能了解基础的 PPT 解析内容，或看我的入门级博客。本文将告诉大家如何从 PPT 里面解析出通用元素的 x 和 y 的值，以及元素的宽度和高度的值

<!--more-->
<!-- CreateTime:2020/3/16 16:35:42 -->



在开始之前请看 [C# dotnet 使用 OpenXml 解析 PPT 文件](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-PPT-%E6%96%87%E4%BB%B6.html) 在拿到 slidePart.Slide.CommonSlideData.ShapeTree 里面的元素，几乎所有元素都存在坐标和宽度高度，这里的元素我称为通用元素，也就是不是特定的如形状、图片元素

此时的元素应该是继承 OpenXmlElement 类，在这个类里面可以通过 GetFirstChild 找到 ShapeProperties 的值

```csharp
            var shapeProperties = element.GetFirstChild<ShapeProperties>();
```

上面代码的 element 是 OpenXmlElement 类

拿到了 ShapeProperties 实际上就是 PPT 文件的 p:sppr 内容，在 PPT 里面将会用如下格式设置元素里面的值中 a:xfrm 就是 Transform2D 请看下面

```xml
<p:sppr>
    <a:xfrm>
        <a:off x="3292475" y="1300390">
        </a:off>
        <a:ext cx="6096000" cy="3429000">
        </a:ext>
    </a:xfrm>
    <a:prstgeom prst="rect">
        <a:avlst>
        </a:avlst>
    </a:prstgeom>
</p:sppr>
```

此时通过 `var transform2D = shapeProperties.GetFirstChild<Drawing.Transform2D>();` 就能拿到 `a:xfrm` 的值

等等，这里的 Drawing.Transform2D 是什么意思，我在命名空间里面添加这个代码

```csharp
using Presentation = DocumentFormat.OpenXml.Presentation;
using Drawing = DocumentFormat.OpenXml.Drawing;
using ShapeProperties = DocumentFormat.OpenXml.Presentation.ShapeProperties;
```

拿到 Transform2D 可以再获取 Offset 也就是 `a:off` 拿到 x 和 y 的大小

```csharp
            var offset = transform2D.GetFirstChild<Drawing.Offset>();
            var offsetX = new Emu(offset.X.Value);
            var offsetY = new Emu(offset.Y.Value);
```

在 PPT 里面，通用元素的 x 和 y 值单位是 Emu 上面的类是我自己定义的，有可以抄的代码，请看 [C# dontet Office Open XML Unit Converter](https://blog.lindexi.com/post/C-dontet-Office-Open-XML-Unit-Converter.html) 我定义了和像素转换的代码

可以通过 Extents 也就是 `a:ext` 获取元素的宽度和高度，请看代码

```csharp
            var extents = transform2D.GetFirstChild<Drawing.Extents>();
            var extentWidth = new Emu(extents.Cx);
            var extentHeight = new Emu(extents.Cy);
```

[ShapeProperties Class (DocumentFormat.OpenXml.Drawing.Pictures)](https://docs.microsoft.com/en-us/dotnet/api/documentformat.openxml.drawing.pictures.shapeproperties?view=openxml-2.8.1 )

知道了元素的坐标如何在 UWP 中设置元素的坐标请看 [win10 uwp 拖动控件](https://blog.lindexi.com/post/win10-uwp-%E6%8B%96%E5%8A%A8%E6%8E%A7%E4%BB%B6.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
