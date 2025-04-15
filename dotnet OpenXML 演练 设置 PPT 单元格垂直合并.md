 # dotnet OpenXML 演练 设置 PPT 单元格垂直合并

本文将告诉大家如何在使用 OpenXML SDK 为

<!--more-->
<!-- CreateTime:2025/04/15 07:19:14 -->

<!-- 草稿 -->



```csharp
using System;
using System.Linq;

using DocumentFormat.OpenXml.Drawing;
using DocumentFormat.OpenXml.Presentation;

using GraphicFrame = DocumentFormat.OpenXml.Presentation.GraphicFrame;
using NonVisualDrawingProperties = DocumentFormat.OpenXml.Presentation.NonVisualDrawingProperties;
using NonVisualShapeProperties = DocumentFormat.OpenXml.Presentation.NonVisualShapeProperties;

using var presentationDocument =
    DocumentFormat.OpenXml.Packaging.PresentationDocument.Open("Test.pptx", true);
var presentationPart = presentationDocument.PresentationPart;
var slidePart = presentationPart!.SlideParts.First();
var slide = slidePart.Slide;

var graphicFrame = slide.CommonSlideData!.ShapeTree!.GetFirstChild<GraphicFrame>()!;
var graphic = graphicFrame.Graphic!;
var graphicData = graphic.GraphicData!;
var table = graphicData.GetFirstChild<Table>()!; // a:tbl
var firstRow = table.Elements<TableRow>().First();
var lastRow = table.Elements<TableRow>().Last();

var firstRowFirstCell = firstRow.Elements<TableCell>().First();
var lastRowFirstCell = lastRow.Elements<TableCell>().First();

firstRowFirstCell.RowSpan = 2;
lastRowFirstCell.VerticalMerge = true;
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/13f2fabf1f08b35d356d88c04cdf4f8a86c4aef0/Pptx/GayurjabeaNiyegacher) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/13f2fabf1f08b35d356d88c04cdf4f8a86c4aef0/Pptx/GayurjabeaNiyegacher) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 13f2fabf1f08b35d356d88c04cdf4f8a86c4aef0
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 13f2fabf1f08b35d356d88c04cdf4f8a86c4aef0
```

获取代码之后，进入 Pptx/GayurjabeaNiyegacher 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )