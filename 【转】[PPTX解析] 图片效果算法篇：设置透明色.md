# 【转】 PPTX解析 设置透明色

在PPT中，我们可以通过图片格式选项中->颜色->设置透明色，将指定颜色设置为透明，以实现去除纯色背景的需求。

<!--more-->
<!-- CreateTime:2021/11/22 9:11:23 -->


![](http://image.acmx.xyz/lindexi%2Fimage-1628836752925.png)
<!-- ![file](https://imxcg.com/wp-content/uploads/2021/08/image-1628836752925.png) -->

原文：[[PPTX解析] 图片效果算法篇：设置透明色 - 仙尘阁](https://imxcg.com/technology/dot-net/pptx-analysis/pptx-analysis-set-transparent-color/ )

## 效果原理

PPT中设置透明色的逻辑十分简单清晰：

> · 将颜色A替换为颜色B

当颜色B为透明色时，即可达成将某一种颜色设置为透明色的要求。


## 解析颜色替换节点（OpenXml）

首先，我们来观察一下PPT存储设置透明色的xml节点：

``` xml
<p:pic>
	......
	<p:blipFill>
		<a:blip ......>
			<a:clrChange>
				<a:clrFrom>
					<a:srgbClr val="FFAAFF" >
						<a:alpha val="100000" />
					</a:srgbClr>
				</a:clrFrom>
				<a:clrTo>
					<a:srgbClr val="FFAAFF">
						<a:alpha val="0" />
					</a:srgbClr>
				</a:clrTo>
			</a:clrChange>
		</a:blip>
		......
	</p:blipFill>
	......
</p:pic>
```

节点属性解析：

|节点名称|含义|值含义|
|-|-|-|
|p:pic|图片|此元素指定文档中的图片对象的存在|
|p:blipFill|图片填充|此元素指定图片对象具有图片填充的类型|
|a:blip|Blip|此元素指定的图像 （二进制大图像或图片） 存在并包含图像数据的引用|
|a:clrChange|变色效果|此元素指定颜色变化效果。 clrFrom 的实例替换为 clrTo 的实例。|
|a:clrFrom|Change Color From|此元素指定在颜色更改效果中移除的颜色。它是“来自”或源输入颜色。|
|a:clrTo|Change Color To|此元素指定替换 clrChange 效果中的 clrFrom 的颜色。这是颜色变化效果中的“目标”或“目标”颜色。|

### 颜色的Alpha通道解析

节点**`<a:srgbClr />`**中存在属性值`val`，该属性表示的是颜色的RGB通道的十六进制表示。

子节点**`<a:alpha />`**中存在属性值`val`，该属性表示的是颜色的Alpha通道的千倍百分比（如：100000实际值代表100%）。

注：当节点**`<a:alpha />`**中的`val`值为100000时，该节点可以省略。即当节点**`<a:srgbClr />`**中不存在子节点**`<a:alpha />`**时，表示该颜色的Alpha通道的百分比为100%(255)。

在该案例中，我们解析出要将图片中的颜色`#FFAAFF`替换成颜色`#00FFAAFF`。

### 效果实现（C#）

可以参考下面的代码，将指定`Bitmap`的颜色进行替换：

``` C#
/// <summary>
///     将图片<paramref name="bitmap"/>上指定的颜色<paramref name="colorA"/>替换为颜色<paramref name="colorB"/>
/// </summary>
/// <param name="bitmap">图片</param>
/// <param name="colorA">要被替换的颜色</param>
/// <param name="colorB">要将<paramref name="colorA"/>替换的成颜色</param>
public void ReplaceColor(Bitmap bitmap, System.Drawing.Color colorA, System.Drawing.Color colorB)
{
    //这里是遍历图片中的每一个像素
    bitmap.PerPixelProcess(color =>
    {
        //如果当前的颜色和颜色colorA近似，则进行替换
        var isSimilar = IsSimilarColors(color, colorA);
        return isSimilar ? colorB : color;
    });
}

/// <summary>
///     是否是近似颜色
/// </summary>
/// <param name="x"></param>
/// <param name="y"></param>
/// <param name="accuracy">Rgb通道允许的误差</param>
/// <returns></returns>
private bool IsSimilarColors(System.Drawing.Color x, System.Drawing.Color y, int accuracy = 36)
{
    var offsetA = x.A - y.A;
    var offsetR = x.R - y.R;
    var offsetG = x.G - y.G;
    var offsetB = x.B - y.B;

    if (Math.Abs(offsetA) > 1)
    {
        return false;
    }

    if (offsetR == offsetG && offsetR == offsetB)
    {
        if (Math.Abs(offsetR) > 1)
        {
            return ColorDifference(x, y) <= accuracy / 3d;
        }
    }

    var difference = ColorDifference(x, y);
    return difference <= accuracy;
}

/// <summary>
/// 在RGB空间上通过公式计算出加权的欧式距离
/// </summary>
/// <param name="x"></param>
/// <param name="y"></param>
/// <returns></returns>
private double ColorDifference(System.Drawing.Color x, System.Drawing.Color y)
{
    var m = (x.R + y.R) / 2.0;
    var r = Math.Pow(x.R - y.R, 2);
    var g = Math.Pow(x.G - y.G, 2);
    var b = Math.Pow(x.B - y.B, 2);

    return Math.Sqrt((2 + m / 256) * r + 4 * g + (2 + (255 - m) / 256) * b);
}
```

### GitHub项目仓库:

如果希望参考完整案例，请参考下面的项目：
[设置透明色案例](https://github.com/Firito/Learnland.DotNetCore.Source/blob/master/Learnland.DotNetCore.PptxAnalysis/Image/ColorEffect.cs "设置透明色案例")
