# 【转】[PPTX解析] 图片 重新着色存储详解

PPTX对图片进行**重新着色**和**设置透明颜色**这两个行为，PPT并不会对原图进行修改，而是通过将修改信息直接存入xml中。也就是说不会生成一张处理过的图片，第三方应用需要获取存储信息，解析后将原图进行修改或通过着色器改变渲染效果。

<!--more-->

<!-- 发布 -->

原作者： 晓嗔戈

## 存储解析

首先，让我们来看一下PPTX提供了哪些**重新着色**的选项：

![file](http://imxcg.com:55555/wp-content/uploads/2021/07/image-1627524289095.png)

**可以看到，PPT的着色选项大体分为以下几类：**

|效果类型|包含的选项|原理|
|-|-|-|
|GrayScaleEffect|灰度|将图片修改为灰阶图|
|ErosionEffect|冲蚀|修改图片的亮度和对比度|
|Black/WhiteEffect|黑白：25%、黑白：50%、黑白：75%、|根据指定阈值将像素设为黑色或白色|
|DuotoneEffect|上述效果外的选项（包含**其他变体**）|对于每个像素，通过线性插值组合 clr1 和 clr2 以确定该像素的新颜色|

**需要注意的是，实际上重新着色是需要根据选择的类型不同，其存储方案也是不同的！（根据规范都是存储在**`<a:blip />`**节点中的）**

``` xml
<p:blipFill>
    <a:blip ...>
        ...
   </a:blip>
    ...
</p:blipFill>
```

## 图像处理矩阵

在讲解如何解析PPTX图像存储前，我们先来说明一下如何处理图片，将其按照我们的想法和输入数据将其进行转换。

在图像处理中，我们通常使用矩阵来进行图像的像素处理，下面是一些常见的C#图像处理矩阵：

``` C#
using System;
using System.Drawing;
using System.Drawing.Drawing2D;
using System.Drawing.Imaging;

namespace ImageProcessService
{
    /// <summary>
    ///     颜色转换矩阵
    /// </summary>
    public static class ColorMatrices
    {
        /// <summary>
        /// 使用给定的数量创建亮度过滤器矩阵。
        /// <para>
        /// 使用算法<see href="https://cs.chromium.org/chromium/src/cc/paint/render_surface_filters.cc"/>
        /// </para>
        /// </summary>
        /// <remarks>
        /// 值为 0 将创建一个完全黑色的图像。值为 1 时输入保持不变。
        /// 其他值是效果的线性乘数。允许超过 1 的值，从而提供更明亮的结果。
        /// </remarks>
        /// <param name="amount">转化比例，必须大于或等于 0。</param>
        /// <returns>The <see cref="ColorMatrix"/>.</returns>
        public static ColorMatrix CreateBrightnessMatrix(float amount)
        {
            if (amount < 0)
            {
                throw new ArgumentOutOfRangeException(nameof(amount), "Threshold must be >= 0");
            }

            return new ColorMatrix
            {
                Matrix00 = amount,
                Matrix11 = amount,
                Matrix22 = amount,
                Matrix33 = 1F
            };
        }

        /// <summary>
        /// 使用给定的数量创建灰度滤波器矩阵。
        /// <para>
        /// 使用算法<see href="https://en.wikipedia.org/wiki/Luma_%28video%29#Rec._601_luma_versus_Rec._709_luma_coefficients"/>
        /// </para>
        /// </summary>
        /// <param name="amount">转化比例，必须大于或等于 0。</param>
        /// <returns>The <see cref="ColorMatrix"/>.</returns>
        public static ColorMatrix CreateGrayScaleMatrix(float amount)
        {
            if (amount < 0 || amount > 1)
            {
                throw new ArgumentOutOfRangeException(
                    nameof(amount),
                    "Threshold must be in range 0..1");
            }

            amount = 1F - amount;

            var matrix = new ColorMatrix();
            matrix.Matrix00 = .299F + (.701F * amount);
            matrix.Matrix10 = .587F - (.587F * amount);
            matrix.Matrix20 = 1F - (matrix.Matrix00 + matrix.Matrix10);

            matrix.Matrix01 = .299F - (.299F * amount);
            matrix.Matrix11 = .587F + (.2848F * amount);
            matrix.Matrix21 = 1F - (matrix.Matrix01 + matrix.Matrix11);

            matrix.Matrix02 = .299F - (.299F * amount);
            matrix.Matrix12 = .587F - (.587F * amount);
            matrix.Matrix22 = 1F - (matrix.Matrix02 + matrix.Matrix12);
            matrix.Matrix33 = 1F;

            return matrix;
        }

        /// <summary>
        /// 使用给定的数量创建对比度过滤器矩阵。
        /// <para>
        /// 使用算法<see href="https://cs.chromium.org/chromium/src/cc/paint/render_surface_filters.cc"/>
        /// </para>
        /// </summary>
        /// <remarks>
        /// 值为 0 将创建一个完全灰色的图像。值为 1 时输入保持不变。
        /// 其他值是效果的线性乘数。允许超过 1 的值，从而提供具有更高对比度的结果。
        /// </remarks>
        /// <param name="amount">转化比例，必须大于或等于 0。</param>
        /// <returns>The <see cref="ColorMatrix"/>.</returns>
        public static ColorMatrix CreateContrastMatrix(float amount)
        {
            if (amount < 0)
            {
                throw new ArgumentOutOfRangeException(nameof(amount), "Threshold must be >= 0");
            }

            var contrast = (-.5F * amount) + .5F;

            return new ColorMatrix
            {
                Matrix00 = amount,
                Matrix11 = amount,
                Matrix22 = amount,
                Matrix33 = 1F,
                Matrix40 = contrast,
                Matrix41 = contrast,
                Matrix42 = contrast
            };
        }

        /// <summary>
        /// 使用给定的数量创建饱和度过滤器矩阵。
        /// <para>
        /// 使用算法<see href="https://cs.chromium.org/chromium/src/cc/paint/render_surface_filters.cc"/>
        /// </para>
        /// </summary>
        /// <remarks>
        /// 0 值是完全不饱和的。值为 1 时输入保持不变。
        /// 其他值是效果的线性乘数。允许超过 1 的值，提供超饱和结果。
        /// </remarks>
        /// <param name="amount">转化比例，必须大于或等于 0。</param>
        /// <returns>The <see cref="ColorMatrix"/>.</returns>
        public static ColorMatrix CreateSaturationMatrix(float amount)
        {
            if (amount < 0)
            {
                throw new ArgumentOutOfRangeException(nameof(amount), "Threshold must be >= 0");
            }

            var matrix = new ColorMatrix();
            matrix.Matrix00 = .213F + (.787F * amount);
            matrix.Matrix10 = .715F - (.715F * amount);
            matrix.Matrix20 = 1F - (matrix.Matrix00 + matrix.Matrix10);

            matrix.Matrix01 = .213F - (.213F * amount);
            matrix.Matrix11 = .715F + (.285F * amount);
            matrix.Matrix21 = 1F - (matrix.Matrix01 + matrix.Matrix11);

            matrix.Matrix02 = .213F - (.213F * amount);
            matrix.Matrix12 = .715F - (.715F * amount);
            matrix.Matrix22 = 1F - (matrix.Matrix02 + matrix.Matrix12);
            matrix.Matrix33 = 1F;

            return matrix;
        }
    }
}

```

在获取到这些处理的矩阵后，我们可以将其应用到图片上，代码如下（C#)：

``` c#
        /// <summary>
        /// 在图像上执行颜色矩阵的应用。
        /// </summary>
        /// <param name="image">图像</param>
        /// <param name="colorMatrix">颜色处理矩阵.</param>
        public static void ApplyMatrix(Image image, ColorMatrix colorMatrix)
        {
            using var graphics = Graphics.FromImage(image);
            using var imageAttributes = new ImageAttributes();
            imageAttributes.SetColorMatrix(colorMatrix, ColorMatrixFlag.Default, ColorAdjustType.Bitmap);

            var imageWidth = image.Width;
            var imageHeight = image.Height;
            var imageRect = new Rectangle(0, 0, imageWidth, imageHeight);
            graphics.CompositingMode = CompositingMode.SourceCopy;
            graphics.DrawImage(image, imageRect, 0, 0, imageWidth, imageHeight, GraphicsUnit.Pixel, imageAttributes);
        }
```

## 存储属性解析

接下来，就让我们逐一的对存储节点进行分析吧~

### GrayScaleEffect

该属性的标记非常简单和清晰，使用了一个单一的节点**`<a:grayscl />`**表明是否应用了该效果:

``` xml
<p:blipFill>
    <a:blip ...>
        <a:grayscl />
        ...
    </a:blip>
    ...
</p:blipFill>
```

所以只要在解析PPTX节点的过程之中，发现存在节点**`<a:grayscl />`**，就可以通过**灰度滤波器矩阵**对图片做灰阶处理即可。

### ErosionEffect

该属性通过节点**`<a:lum />`**进行存储，如下:

``` xml
<p:blipFill>
    <a:blip ...>
        <a:lum bright="70000" contrast="-70000" />
        ...
    </a:blip>
    ...
</p:blipFill>
```

该节点属性解析：

|属性名称|属性含义|值含义|补充说明|
|-|-|--|-|
|bright|亮度|正值表示增加亮度，负值表示降低亮度|单位：千倍百分比（如：70000实际值代表70%）|
|contrast|对比度|正值表示增加对比度，负值表示降低对比度|单位：千倍百分比（如：-70000实际值代表-70%）|

所以只要在解析PPTX节点的过程之中，发现存在节点**`<a:lum />`**，就可以通过**亮度滤波器矩阵**和**对比度滤波器矩阵**对图片做处理即可。当然，如果学习过线性代数，那么你应该是知道可以通过矩阵叉获得一个连续转换矩阵的吧~ 不要傻乎乎的真的分别将图片修改一次亮度，然后再修改一次对比度呦~

**效果不一致问题：**

在实际转换后，你可能会发现：应用亮度和对比度转换时，当亮度和对比度修改值为正时，PPT的效果和运用转换矩阵后的效果强度是不同的，并且PPT在设置亮度为最大值bright="100000"时，图片是全白的！
这是因为PPT运用的修改亮度算法和对比度算法与我们运用的算法是不同的（算法好多，而且还可以魔改）。这里我们通过一个简单的算法，得到一个转化比例，让转换效果基本和PPT保持一致一致，代码如下（C#）：

``` C#
        /// <summary>
        /// 获取和PPT效果近似的转化比例（0~69内使用2次方，70~79内使用3次方，80~89内使用4次方，90~99内使用5次方）
        /// </summary>
        /// <param name="percentage"></param>
        /// <returns></returns>
        private float GetNearlyAmount(float percentage)
        {
            var amount = percentage + 1;
            if (percentage > 0)
            {
                var x = (percentage - 60) / 10;
                var y = 2 + (x > 0 ? x : 0);
                amount = (float)Math.Pow(amount, y);
            }

            return amount;
        }
```

例如，我们在处理亮度时，如果解析出bright=70000，那么根据千倍百分比可以得出bright实际效果值应该为0.7，我们将0.7传入**GetNearlyAmount()**即可获得一个和PPT亮度修改效果近似的转化比例。接下来调用CreateBrightnessMatrix()，就可以获得亮度转换矩阵了。（注意：当bright=100000时，我们直接创建一个纯白的图即可）。

### Black/WhiteEffect

该属性通过节点**`<a:biLevel />`**进行存储，如下:

``` xml
<p:blipFill>
    <a:blip ...>
        <a:biLevel thresh="50000" />
        ...
    </a:blip>
    ...
</p:blipFill>
```

该节点属性解析：

|属性名称|属性含义|值含义|补充说明|
|-|-|--|-|
|thresh|阈值|像素点的亮度大于或等于给定的阈值将显示白色，否则显示黑色|单位：千倍百分比（如：50000实际值代表50%）|

注：在.NET中，我们可以通过`Color.GetBrightness()`获取像素亮度。如何设置像素颜色的方法有很多，在此就不进行叙述了。

### DuotoneEffect

这个效果是PPT**重新着色**中最复杂的效果。
该属性通过节点**`<a:duotone />`**进行存储，如下:

``` xml
<p:blipFill>
    <a:blip ...>
        <a:duotone>
            <a:prstClr val="black" />
            <a:srgbClr val="8D71F5">
                <a:tint val="45000" />
                <a:satMod val="400000" />
            </a:srgbClr>
        </a:duotone>
        ...
    </a:blip>
    ...
</p:blipFill>
```

**`<a:duotone />`**节点解析：

|子节点名称|含义|值含义|补充说明|
|-|-|--|-|
|a:prstClr|预设颜色|指定实际的预设颜色值|实际上就是一个颜色，我们称其为A|
|a:srgbClr|RGB颜色|实际颜色值，表示为十六进制数字 RRGGBB 的序列|实际上就是一个颜色，我们称其为B|

**`<a:srgbClr />`**节点解析：

|子节点名称|含义|值含义|补充说明|
|-|-|--|-|
|a:tint|色调|指定以百分比值表示的色调。|单位：千倍百分比（如：50000实际值代表50%）|
|a:satMod|饱和度调制|指定输入颜色，其饱和度按给定百分比进行调制。|单位：千倍百分比 （50%将饱和度降低一半，200%使饱和度加倍）|

**实践出真知**
看过上面的解析，是不是觉得它的解析很复杂，请忘掉上面的内容！看下面经过实践验证的解析：

经过实践验证，节点**`<a:duotone />`**支持并实际存放的是两个颜色，下面称为`A`,`B`

``` xml
<p:blipFill>
    <a:blip ...>
        <a:duotone>
             <颜色>A</颜色>
             <颜色>B</颜色>
        </a:duotone>
        ...
    </a:blip>
    ...
</p:blipFill>
```

下面我们使用色相环去检测不同顺序和颜色对最终效果的影响：
![image](http://imxcg.com:55555/wp-content/uploads/2021/07/image-1627557433281-e1627557916995.png)

|序列|颜色A|颜色B|结果|
|-|-|-|-|
|1|白|白|全白|
|2|黑|黑|全黑|
|3|红|红|全红|
|4|白|黑|![file](http://imxcg.com:55555/wp-content/uploads/2021/07/image-1627559028516.png)|
|5|黑|白|![file](http://imxcg.com:55555/wp-content/uploads/2021/07/image-1627559411853.png)|
|6|白|红|![file](http://imxcg.com:55555/wp-content/uploads/2021/07/image-1627559235165.png)|
|7|红|白|![file](http://imxcg.com:55555/wp-content/uploads/2021/07/image-1627559361841.png)|
|8|黑|红|![file](http://imxcg.com:55555/wp-content/uploads/2021/07/image-1627559503911.png)|
|9|红|黑|![file](http://imxcg.com:55555/wp-content/uploads/2021/07/image-1627559540591.png)|

## 小知识

> RGB 转为灰度值的心理学公式 Gray = 0.30R + 0.59G + 0.11B
