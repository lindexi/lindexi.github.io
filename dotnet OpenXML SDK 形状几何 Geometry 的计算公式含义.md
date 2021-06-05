# dotnet OpenXML SDK 形状几何 Geometry 的计算公式含义

本文来告诉大家，在 OpenXML 里面的 Geometry 的如 gdLst 和 ahLst 和 pathLst 等里面参数的公式的参数含义

<!--more-->
<!-- CreateTime:2021/6/3 19:41:03 -->

<!-- 发布 -->

这部分内容放在 ECMA-376 的 20.1.10.55 章文档里面，本文只是将文档里面的内容翻译一下

在使用 OpenXML 读取形状时，会看到有些形状的定义内容如下

```xml
    <avLst xmlns="http://schemas.openxmlformats.org/drawingml/2006/main">
      <gd name="adj1" fmla="val 50000" />
    </avLst>
    <gdLst xmlns="http://schemas.openxmlformats.org/drawingml/2006/main">
      <gd name="x2" fmla="*/ w adj1 100000" />
      <gd name="x1" fmla="+/ l x2 2" />
      <gd name="x3" fmla="+/ r x2 2" />
      <gd name="y3" fmla="*/ h 3 4" />
    </gdLst>
    <ahLst xmlns="http://schemas.openxmlformats.org/drawingml/2006/main">
      <ahXY gdRefX="adj1" minX="-2147483647" maxX="2147483647">
        <pos x="x2" y="vc" />
      </ahXY>
    </ahLst>
    <pathLst xmlns="http://schemas.openxmlformats.org/drawingml/2006/main">
      <path fill="none">
        <moveTo>
          <pt x="l" y="t" />
        </moveTo>
        <cubicBezTo>
          <pt x="x1" y="t" />
          <pt x="x2" y="hd4" />
          <pt x="x2" y="vc" />
        </cubicBezTo>
        <cubicBezTo>
          <pt x="x2" y="y3" />
          <pt x="x3" y="b" />
          <pt x="r" y="b" />
        </cubicBezTo>
      </path>
    </pathLst>
```

如果想要绘制形状的 Path 几何图形，就需要了解此形状里面的 Path 的各个值。如上图，可以看到都采用的是公式的方式进行计算，如 gd 的内容如下

```xml
      <gd name="adj1" fmla="val 50000" />
```

以上表示了在 avLst 也就是 AdjustValueList 调整点的参数，以上的 gd 也就是 OpenXML SDK 的 ShapeGuide 类型，这里面的 name 就是 adj1 换句话说就是变量名为 adj1 的值。此 adj1 变量将会在接下来的公式里面使用。而 fmla 就是 ShapeGuide 的 Formula 公式内容，通过如下代码可以获取到公式

```csharp
        private void Foo(ShapeGuide shapeGuide)
        {
            var formula = shapeGuide.Formula;
        }
```

可以看到以上的 `val 50000` 字符串就是公式的内容，以上的 val 表示常量，也就是相当于 `adj1 = val 50000` 也就是 adj1 变量的值就是 50000 的常量

而后续在 gdLst 也就是 ShapeGuideList 类型里面，将会在如下代码使用到 adj1 变量

```xml
    <gdLst xmlns="http://schemas.openxmlformats.org/drawingml/2006/main">
      <gd name="x2" fmla="*/ w adj1 100000" />
      <gd name="x1" fmla="+/ l x2 2" />
      <gd name="x3" fmla="+/ r x2 2" />
      <gd name="y3" fmla="*/ h 3 4" />
    </gdLst>
```

此时在 `gd name="x2" fmla="*/ w adj1 100000"` 里面通过计算，拿到 x2 变量的值，以上使用了 `*/` 这个符号，其实在 OpenXML 里面的公式用的是逆波兰表达的公式，大概的意思就是 `*/` 运算符要求后续传入三个参数，假定这三个参数是 a b c 三个，那么计算的方法是 `(a * b) / c` 拿到值

通过不断代入公式可以拿到对应的变量，从而计算出 Path 里面的内容。但以上有一部分公式使用了常量，如下面代码

```xml
        <moveTo>
          <pt x="l" y="t" />
        </moveTo>
```

上面代码的 `pt x="l" y="t"` 的 l 和 t 都是常量，在文档里面都有定义

下面将告诉大家计算的符号的含义，以及常量的值

## 3cd4

表示三分之四的圆，以上的 c 就是 Circle 圆的意思，而 d 就是除法的意思， 相当于 `3 * 圆 / 4` 的值

以上的圆使用的是 180° 的表示，也就是以上常量的值等于 `3cd4 = 3 x 360° / 4 = 270°` 通过 [Office Open XML 的测量单位](https://blog.lindexi.com/post/Office-Open-XML-%E7%9A%84%E6%B5%8B%E9%87%8F%E5%8D%95%E4%BD%8D.html ) 可以拿到角度对应的值是 16200000.0 的常量值。在 OpenXML 里面使用 60000 表示 360° 的圆

以此可以了解到以下的对圆的计算值

```
3cd4 = 3 x 360° / 4 = 270° = 16200000 Degree
3cd8 = 3 x 360° / 8 = 135° = 8100000 Degree
5cd8 = 5 x 360° / 8 = 225° = 13500000 Degree
7cd8 = 7 x 360° / 8 = 315° = 18900000 Degree
cd2 = 360° / 2 = 180° = 10800000 Degree
cd4 = 360° / 4 = 90° = 5400000 Degree
cd8 = 360° / 8 = 45° = 2700000 Degree
```

## t

也就是 Shape Top Edge 的含义，表示上边缘，等价于常量 0 的值。原因是 OpenXML 的形状采用的坐标系和 DirectX 的坐标系相同，左上角是 0，0 点，从上到下 y 的值不断加大。从左到右 x 的值加大

## b

也就是 Shape Bottom Edge 的含义，等价于常量 h 的值

这是形状的下边缘，因为形状的上边缘被认为是 0 点，因此下边缘就是形状的高度

关于常量 h 的值，请看下文

## h

也就是 Shape Height 的含义，表示形状的高度，需要通过形状的属性拿到形状的高度才能了解此值

## hd2

表示的是高度除以 2 的值，以上的 h 是 高度 而 d 表示的是除以，相当于如下公式

```
*/ h 1.0 2.0
```

以此可以了解如下的几个常量的计算

```
hd2 = */ h 1.0 2.0 = height / 2
hd4 = */ h 1.0 4.0 = height / 4
hd5 = */ h 1.0 5.0 = height / 5
hd6 = */ h 1.0 6.0 = height / 6
hd8 = */ h 1.0 8.0 = height / 8
```

## vc

也就是 Vertical Center of Shape 的含义，表示垂直的中心，相当于高度的一半，使用如下公式

```
*/ h 1.0 2.0
```

以上代码的 `*/` 公式内容请参阅下文，而 h 表示的是宽度


## l

也就是 Shape Left Edge 的含义，表示左边缘的值，等价于常量 0 的值。原因是 OpenXML 的形状采用的坐标系和 DirectX 的坐标系相同，左上角是 0，0 点，从上到下 y 的值不断加大。从左到右 x 的值加大

## r

也就是 Shape Right Edge 的含义，表示右边缘的值，等价于常量 w 的值。也就是右边缘的值和形状的宽度相同，因为形状的左边缘是 0 的值，因此形状的右边的值就和形状的宽度相同

关于 w 请看下文

## w

也就是 Shape Width 形状宽度的含义，需要通过形状的属性拿到形状的高度才能了解此值

## wd2

表示形状宽度的一半，以上的 w 是 宽度 而 d 表示的是除以，相当于如下公式

```
*/ w 1.0 2.0
```

以此可以了解如下的几个常量的计算

```
wd2 = */ w 1.0 2.0 = width / 2
wd4 = */ w 1.0 4.0 = width / 4
wd5 = */ w 1.0 5.0 = width / 5
wd6 = */ w 1.0 6.0 = width / 6
wd8 = */ w 1.0 8.0 = width / 8
wd10 = */ w 1.0 10.0 = width / 10
```

## hc

也就是 Horizontal Center 的含义，表示水平的中心点，相当于宽度的一半，计算的公式如下

```
*/ w 1.0 2.0
```

以上代码的 `*/` 公式内容请参阅下文，而 w 表示的是宽度

## ls

也就是 Longest Side of Shape 的含义，表示宽度或高度里面最长的一边，等价以下公式

```
max w h
```

也就是返回宽度或高度的最大值

## ss

也就是 Shortest Side of Shape 的含义，表示宽度或高度里面最短的一边，等价以下公式

```
min w h
```

也就是返回宽度或高度的最小值

## ssd2

表示的是 ss 除以 2 的值，也就是获取宽度或高度的最小值除以 2 的值，以上 d 表示的是除以，使用如下公式

```
*/ ss 1.0 2.0
```

以此可以了解如下的几个常量的计算

```
ssd2 = */ ss 1.0 2.0 = Shortest Side / 2
ssd4 = */ ss 1.0 4.0 = Shortest Side / 4
ssd6 = */ ss 1.0 6.0 = Shortest Side / 6
ssd8 = */ ss 1.0 8.0 = Shortest Side / 8
```

## 符号

而形状的计算符号定义在 ECMA 376 的 20.1.9.11 章文档

含义如下，以下的 x 和 y 和 z 表示传入的三个参数的值，如 `fmla="*/ x y z"` 的实际文档的值是 `fmla="*/ 1 2 3"` 也就是表示 x = 1 ，y = 2 ，z = 3 的值

## Multiply Divide Formula

乘除公式使用 `*/` 表示，要求传入三个参数

```
"*/ x y z" = ((x * y) / z)
```

## Add Subtract Formula

加减公式使用 `+-` 表示，要求传入三个参数

```
"+- x y z" = ((x + y) - z)
```

## Add Divide Formula

加除公式使用 `+/` 表示，要求传入三个参数

```
"+/ x y z" = ((x + y) / z)
```

## If Else Formula

条件判断使用 `?:` 符号表示，和 C# 里面的 `?:` 逻辑相同，需要传入三个参数，假定参数是 x y z 三个参数，判断是如果传入的 x 大于 0 那么则是 true 代码如下

```
"?: x y z" = x > 0 ? y : z

if (x > 0)
{
        return y;
}
else
{
        return z;
}
```

## Absolute Value Formula

绝对值公式使用 `abs` 表示，需要传入一个参数，计算方法如下

```
abs x = Math.Abs(x)
```

## ArcTan Formula

表示三角函数的 `arctan2` 公式，计算方法如下

```
at2 x y = arctan(y / x) = Math.Atan2(y, x)
```

## Cosine ArcTan Formula

表示三角函数的两次计算 `cat2` 公式，计算方法如下

```
cat2 x y z = (x*(cos(arctan(z / y))) = (x * Math.Cos(Math.Atan2(z, y)))
```

## Cosine Formula

表示三角函数的 `cos` 公式，计算方法如下

```
cos x y = (x * cos( y )) = (x * Math.Cos(y))
```

## Sine ArcTan Formula

表示三角函数的 `sat2` 公式，计算方法如下

```
sat2 x y z = (x*sin(arctan(z / y))) = (x * Math.Sin(Math.Atan2(z, y)))
```

## Sine Formula

表示三角函数的 `sin` 公式，计算方法如下

```
sin x y = (x * sin( y )) = (x * Math.Sin(y))
```

## Tangent Formula

表示三角函数的 `tan` 公式，计算方法如下

```
tan x y = (x * tan( y )) = (x * Math.Tan(y))
```

## Maximum Value Formula

表示两个数里面最大的一个值，使用 `max` 公式，计算方法如下

```
max x y = Math.Max(x, y)
```

## Minimum Value Formula

表示两个数里面最小的一个值，使用 `min` 公式，计算方法如下

```
min x y = Math.Min(x, y)
```

## Modulo Formula

表示 `mod` 公式，计算方法如下

```
mod x y z = sqrt(x^2 + b^2 + c^2) = Math.Sqrt(x * x + y * y + z * z)
```

## Pin To Formula

表示 `pin` 公式，计算方法如下

```
pin x y z = 

if (y < x)
{
        return x;
}
else if (y > z)
{
        return z;
}
else
{
        return y;
}
```

## Square Root Formula

表示 `sqrt` 公式，计算方法如下

```
sqrt x = Math.Sqrt(x)
```

## Literal Value Formula

表示一个常量的值，相当于 `var` 的定义，表示的是 `val` 公式，将会返回对应的值

如 `val x` 就是返回 x 的值

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

<!-- 


ECMA 376 3251 fmla (Shape Guide Formula)

abs = abs 绝对值

at2 = arctan2 三角函数

cat2 = cos(arctan2(x)) 三角函数

cos = cos(x) 三角函数

pin = pin(a, b, c)
{
        if (a > b)
        {
                return a;
        }
        else if(b > c)
        {
                return c;
        }
        else
        {
                return b;
        }
}

sat2 = sin(arctan2(x)) 三角函数

sin = sin(x) 三角函数

sqrt = sqrt(x)

tan = tan(x) 三角函数



[presetShapeDefinitions.xml does not specify width and height form some autoshapes](https://social.msdn.microsoft.com/Forums/en-US/3f69ebb3-62a0-4fdd-b367-64790dfb2491/presetshapedefinitionsxml-does-not-specify-width-and-height-form-some-autoshapes?forum=os_binaryfile )


20.1.10.55 ST_ShapeType (Preset Shape Types)

Built-in Guides  Comment

3cd4 = 3 x 360° / 4 = 270°

3cd8 = 3 x 360° / 8 = 135°

5cd8 = 5 x 360° / 8  =  225°

7cd8 = 7 x 360° / 8  =  315°

b = bottom

cd2 = 360° / 2  =  180°

cd4 = 360° / 4  =  90°

cd8 = 360° / 8  =  45°

hc = horizontal center

h = height

hd2 = height / 2

hd4 = height / 4

hd5 = height / 5

hd6 = height / 6

hd8 = height / 8

l = left

ls = long side

r = right

ss = short side

ssd2 = short side / 2

ssd4 = short side / 4

ssd6 = short side / 6

ssd8 = short side / 8

t = top

vc = vertical center

w = width

wd2 = width / 2

wd4 = width / 4

wd5 = width / 5

wd6 = width / 6

wd8 = width / 8

wd10 = width / 10


 -->
