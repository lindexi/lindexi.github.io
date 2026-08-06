# WPF 使用像素着色器实现单应变换

本文将告诉大家如何在 WPF 中通过自定义 ShaderEffect 像素着色器实现图片的单应变换，也就是透视变换效果，拖动图片四个角点可以实时调整变换形态，整个过程由 GPU 逐像素完成，Demo 中附带了凸四边形校验，避免把图片拖成无法变换的形态

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

本文内容由人类主导 AI 辅助编写

## 背景

单应变换（Homography）是计算机视觉里很常用的变换，它的特点是能把一个任意四边形映射到另一个任意四边形，比如把一张贴纸图片投影到照片里的墙面、把倾斜拍摄的文档矫正成正视图，都属于单应变换的应用。数学上它由一个 3x3 的矩阵描述，和普通的仿射变换不同，这个矩阵还带有透视分量，因此能把平行线变成相交线，产生近大远小的效果

在 WPF 里面想要做出这种透视效果，常见的选择是用 PlaneProjection 或者 3D 场景里的相机投影。但这两个方案本质上都是把平面放到空间里旋转，能模拟的形态有限，而且想要"任意四边形到任意四边形"的自由映射并不方便。另一个思路是使用像素着色器：每个输出像素都去计算自己对应源纹理的哪个位置，然后把颜色取过来，这是完全自由的，只要数学算得出来，什么变换都能做

WPF 原生支持自定义 ShaderEffect，只需要把编译好的像素着色器字节码交给 PixelShader 即可。本文的 Demo 就是一个完整的可运行示例，界面如下：中间是图片，四角有黄色圆环手柄，拖动手柄图片实时变形，虚线框标出当前四边形，四边形之外的区域保持棋盘格背景

## 整体流程

先看一下整个 Demo 的工作流程，方便理解后面每一段代码所处的位置：

1. 在 XAML 里放置图片、棋盘格背景、四边形轮廓和四个拖拽手柄
2. 拖拽手柄时，把拖动量换算成归一化的角点坐标，更新角点数组
3. 校验四个角点是否构成凸四边形，不合法则回滚
4. 用角点计算"四边形到单位正方形"的逆单应矩阵
5. 把矩阵三行传给 ShaderEffect 的常量寄存器
6. GPU 运行像素着色器，逐像素从源纹理取色完成变换

其中第 4 步的矩阵计算是整个 Demo 的核心，第 6 步的着色器是效果的关键，拖拽交互只是把这两者串起来

## 角点的坐标系约定

打开 MainWindow.xaml.cs，可以看到角点数据保存在 `_corners` 数组里，四个点的顺序是左上、右上、右下、左下，也就是顺时针：

```csharp
private readonly Point[] _corners =
[
    new(0.08, 0.10),
    new(0.78, 0.04),
    new(0.94, 0.82),
    new(0.12, 0.94)
];
```

这里非常关键的一点是：坐标全部归一化到 0 到 1 之间，而不是直接使用像素值。为什么要这么做？因为单应矩阵的数学推导是基于单位正方形（0 到 1 的坐标系）的，归一化之后，无论窗口怎么缩放、Viewport 是多大，矩阵计算都不受影响，只在最后把坐标乘回 Viewport 的宽高用于绘制。这个解耦让整个 Demo 简单了很多

## 单应矩阵的计算

单应变换的矩阵计算分为两步：先算"单位正方形到四边形"的正向矩阵，再对它求逆得到"四边形到单位正方形"的逆矩阵。代码里 `Homography` 静态类封装了全部数学逻辑

先看正向矩阵 `CreateUnitSquareToQuadrilateral`，它接收四个角点，返回一个 3x3 矩阵：

```csharp
private static double[,] CreateUnitSquareToQuadrilateral(
    Point topLeft,
    Point topRight,
    Point bottomRight,
    Point bottomLeft)
{
    var dx1 = topRight.X - bottomRight.X;
    var dx2 = bottomLeft.X - bottomRight.X;
    var dx3 = topLeft.X - topRight.X + bottomRight.X - bottomLeft.X;
    var dy1 = topRight.Y - bottomRight.Y;
    var dy2 = bottomLeft.Y - bottomRight.Y;
    var dy3 = topLeft.Y - topRight.Y + bottomRight.Y - bottomLeft.Y;
    var denominator = (dx1 * dy2) - (dx2 * dy1);

    if (Math.Abs(denominator) < Epsilon)
    {
        throw new ArgumentException("四边形无法构成有效的单应变换。");
    }

    var g = ((dx3 * dy2) - (dx2 * dy3)) / denominator;
    var h = ((dx1 * dy3) - (dx3 * dy1)) / denominator;

    return new[,]
    {
        { topRight.X - topLeft.X + (g * topRight.X), bottomLeft.X - topLeft.X + (h * bottomLeft.X), topLeft.X },
        { topRight.Y - topLeft.Y + (g * topRight.Y), bottomLeft.Y - topLeft.Y + (h * bottomLeft.Y), topLeft.Y },
        { g, h, 1 }
    };
}
```

这段代码看起来有点吓人，其实它是在解一个线性方程组。单应矩阵的第三行是 `(g, h, 1)`，其中的 `g` 和 `h` 就是透视分量，它们决定了变换的"倾斜"程度。`dx1`、`dx2`、`dx3` 这些中间量分别是四边形的对角差和"歪斜度"（topLeft - topRight + bottomRight - bottomLeft 衡量的是四边形偏离平行四边形的程度），代入公式就能解出 `g` 和 `h`

这个矩阵的计算方式不是原创的，而是来自 OpenCV 的 `getPerspectiveTransform` 实现，是图像处理领域非常经典的公式，直接把四个目标角点代进去就能得到映射矩阵

拿到 `g` 和 `h` 之后，矩阵的前两行也就定了。`denominator` 是分母，如果它接近 0，说明四个点退化成了三角形或者共线，这时候矩阵无解，直接抛异常告诉调用方四边形不合法

有了正向矩阵之后，为什么还需要求逆？这就要回到像素着色器的工作方式了。着色器是对**输出图像**的每个像素执行的：它知道目标像素的坐标（destination），但不知道这个位置应该取源纹理的哪个颜色。所以我们需要的是反过来的映射，也就是"输出图像上的点，对应源纹理的哪个位置"，这就是 `CreateQuadrilateralToUnitSquare` 要做的事：

```csharp
public static double[,] CreateQuadrilateralToUnitSquare(IReadOnlyList<Point> corners)
{
    ArgumentNullException.ThrowIfNull(corners);

    if (corners.Count != 4)
    {
        throw new ArgumentException("必须提供四个角点。", nameof(corners));
    }

    var forward = CreateUnitSquareToQuadrilateral(
        corners[0], corners[1], corners[2], corners[3]);
    return Invert(forward);
}
```

它的实现很朴素：先算正向矩阵，再调用 `Invert` 求逆。求逆用的是 3x3 矩阵的伴随矩阵法，对于 3x3 这种小矩阵，直接按公式展开比通用高斯消元更快也更直观：

```csharp
private static double[,] Invert(double[,] matrix)
{
    var a = matrix[0, 0];
    var b = matrix[0, 1];
    var c = matrix[0, 2];
    var d = matrix[1, 0];
    var e = matrix[1, 1];
    var f = matrix[1, 2];
    var g = matrix[2, 0];
    var h = matrix[2, 1];
    var i = matrix[2, 2];
    var determinant = (a * ((e * i) - (f * h)))
                      - (b * ((d * i) - (f * g)))
                      + (c * ((d * h) - (e * g)));

    if (Math.Abs(determinant) < Epsilon)
    {
        throw new InvalidOperationException("单应矩阵不可逆。");
    }

    var scale = 1 / determinant;
    return new[,]
    {
        { ((e * i) - (f * h)) * scale, ((c * h) - (b * i)) * scale, ((b * f) - (c * e)) * scale },
        { ((f * g) - (d * i)) * scale, ((a * i) - (c * g)) * scale, ((c * d) - (a * f)) * scale },
        { ((d * h) - (e * g)) * scale, ((b * g) - (a * h)) * scale, ((a * e) - (b * d)) * scale }
    };
}
```

行列式接近 0 说明矩阵奇异，也就是变换会压扁成一条线，此时同样抛异常。这里的 `Epsilon` 是 `1e-8`，一个足够小的阈值，用来判断浮点数是否为 0，避免直接比较 `== 0` 的浮点精度问题

## 凸四边形校验

矩阵算好了，但如果用户把某个角点拖过头，四个点可能不再构成凸四边形，此时单应变换会得到扭曲甚至翻转的怪异结果。所以在拖拽过程中要实时校验，不合法就回滚。这就是 `IsConvexQuadrilateral` 的作用：

```csharp
public static bool IsConvexQuadrilateral(IReadOnlyList<Point> corners)
{
    ArgumentNullException.ThrowIfNull(corners);

    if (corners.Count != 4)
    {
        return false;
    }

    double? sign = null;
    for (var i = 0; i < corners.Count; i++)
    {
        var first = corners[i];
        var second = corners[(i + 1) % corners.Count];
        var third = corners[(i + 2) % corners.Count];
        var cross = ((second.X - first.X) * (third.Y - second.Y))
                    - ((second.Y - first.Y) * (third.X - second.X));

        if (Math.Abs(cross) < 0.002)
        {
            return false;
        }

        var currentSign = Math.Sign(cross);
        sign ??= currentSign;
        if (sign != currentSign)
        {
            return false;
        }
    }

    return true;
}
```

判断凸四边形的原理很简单：遍历每条边，计算相邻两条边向量的叉积。叉积的符号表示转动的方向，如果所有边的转动方向一致，也就是叉积符号全部相同，那这个四边形就是凸的。`(i + 1) % corners.Count` 和 `(i + 2) % corners.Count` 是取模运算，让最后一个点能绕回第一个点，这样四条边都能遍历到

叉积绝对值小于 `0.002` 时直接判定不合法，这表示两条边几乎共线，四边形退化成了接近三角形或者自交的形态，虽然数学上还能算出矩阵，但视觉效果会很奇怪。这个阈值可以根据手感调整，太大会让角点拖到边缘就拖不动，太小则容易拖出怪异的形状

## 像素着色器

数学部分完成之后，就轮到真正的核心：像素着色器。整个 HLSL 源码就内嵌在 C# 的字符串里，这也是这个 Demo 最大的亮点，先看着色器代码：

```hlsl
sampler2D InputTexture : register(s0);
float4 MatrixRow0 : register(c0);
float4 MatrixRow1 : register(c1);
float4 MatrixRow2 : register(c2);

float4 main(float2 destination : TEXCOORD0) : COLOR0
{
    float3 homogeneousPosition = float3(destination, 1.0);
    float sourceW = dot(MatrixRow2.xyz, homogeneousPosition);
    float2 source = float2(
        dot(MatrixRow0.xyz, homogeneousPosition),
        dot(MatrixRow1.xyz, homogeneousPosition)) / sourceW;
    float2 lowerBound = step(float2(0.0, 0.0), source);
    float2 upperBound = step(source, float2(1.0, 1.0));
    float visible = lowerBound.x * lowerBound.y * upperBound.x * upperBound.y;
    return tex2D(InputTexture, saturate(source)) * visible;
}
```

逐行解释一下。第一行声明了输入采样器 `InputTexture` 绑定到 `s0` 寄存器，这是 WPF 传入的源图像。接下来三行把矩阵的三行声明为 `float4` 常量，绑定到 `c0`、`c1`、`c2` 寄存器，WPF 会通过常量寄存器把 C# 侧的矩阵数据传进来

`main` 函数的输入 `destination` 是目标像素坐标，WPF 渲染时会把它归一化到 0 到 1 范围。把它补上齐次坐标的第三个分量 1.0，就得到 `homogeneousPosition`

接下来的三行就是把逆矩阵应用到坐标上：

```hlsl
float sourceW = dot(MatrixRow2.xyz, homogeneousPosition);
float2 source = float2(
    dot(MatrixRow0.xyz, homogeneousPosition),
    dot(MatrixRow1.xyz, homogeneousPosition)) / sourceW;
```

矩阵乘齐次坐标之后得到一个三维向量，前两个分量是未归一化的源坐标，第三行算出的 `sourceW` 是透视分量，用它做除法，把齐次坐标还原成普通的二维坐标，这就是透视除法。`source` 就是当前输出像素对应的源纹理位置

最后几行处理越界问题：

```hlsl
float2 lowerBound = step(float2(0.0, 0.0), source);
float2 upperBound = step(source, float2(1.0, 1.0));
float visible = lowerBound.x * lowerBound.y * upperBound.x * upperBound.y;
return tex2D(InputTexture, saturate(source)) * visible;
```

当 `source` 落在 0 到 1 之外时，说明这个输出像素在目标四边形外面，那里不应该显示图片内容。`step(a, b)` 在 `b >= a` 时返回 1，否则返回 0，四个边界条件乘起来，落在四边形内的像素 `visible` 为 1，在外的为 0。最后用 `saturate(source)` 把采样坐标夹到合法范围避免越界采样，再乘上 `visible`，四边形外的像素 alpha 变成 0，露出下面的棋盘格背景

注意这里选用的着色器模型是 `ps_2_0`，这是 WPF ShaderEffect 支持的传统着色器模型，指令数有限，但应付这个效果绰绰有余

## 运行时编译着色器

传统 WPF 的 ShaderEffect 用法是先用 fxc.exe 把 .fx 文件预编译成 .ps 字节码文件，作为资源嵌入程序集，再通过 `PixelShader` 的 `UriSource` 加载。这个流程需要额外的构建步骤和文件管理，比较繁琐

这个 Demo 换了一种思路：把 HLSL 源码直接写成 C# 字符串，程序启动时调用 DirectX 的 `D3DCompile` 在运行时编译。这样整个项目不需要任何 .fx 文件，也不需要预编译步骤，代码即着色器，自包含且容易修改。`RuntimePixelShaderCompiler` 类封装了这一切

`D3DCompile` 是 d3dcompiler_47.dll 导出的函数，通过 P/Invoke 调用：

```csharp
[DllImport("d3dcompiler_47.dll", CharSet = CharSet.Ansi)]
private static extern int D3DCompile(
    byte[] sourceData,
    nuint sourceDataSize,
    string sourceName,
    IntPtr defines,
    IntPtr include,
    string entryPoint,
    string target,
    uint flags1,
    uint flags2,
    [MarshalAs(UnmanagedType.Interface)] out ID3DBlob? code,
    [MarshalAs(UnmanagedType.Interface)] out ID3DBlob? errors);
```

d3dcompiler_47.dll 在 Windows 8.1 及之后的系统都自带，Win10 和 Win11 无需额外安装。`entryPoint` 传 `"main"`，`target` 传 `"ps_2_0"`，编译成功后字节码写入 `code` 指向的 `ID3DBlob`，失败时错误信息写入 `errors`

`ID3DBlob` 是 DirectX 的 COM 接口，用于存放编译产物，但它在 C# 里没有现成的互操作定义，需要自己声明。整个接口只有两个方法，取数据指针和取数据大小：

```csharp
[ComImport]
[Guid("8BA5FB08-5195-40E2-AC58-0D989C3A0102")]
[InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
private interface ID3DBlob
{
    [PreserveSig]
    IntPtr GetBufferPointer();

    [PreserveSig]
    nuint GetBufferSize();
}
```

`[ComImport]` 和 `[Guid]` 告诉运行时这是一个 COM 接口，`[PreserveSig]` 表示方法直接返回原始值而不是按 HRESULT 约定包装。这是 COM 互操作里比较容易踩坑的地方，少了 `[PreserveSig]` 运行时会试图把返回值当作 HRESULT 处理，导致拿不到正确的数据指针

编译的主流程如下：

```csharp
public static PixelShader Compile()
{
    var source = Encoding.UTF8.GetBytes(ShaderSource);
    var result = D3DCompile(
        source, (nuint)source.Length, "PerspectiveEffect.fx", IntPtr.Zero, IntPtr.Zero,
        "main", "ps_2_0", 0, 0, out var shaderBlob, out var errorBlob);

    try
    {
        if (result < 0 || shaderBlob is null)
        {
            var message = errorBlob is null
                ? $"D3DCompile 失败，HRESULT: 0x{result:X8}。"
                : ReadBlobText(errorBlob);
            throw new InvalidOperationException(message, Marshal.GetExceptionForHR(result));
        }

        var byteCode = new byte[checked((int)shaderBlob.GetBufferSize())];
        Marshal.Copy(shaderBlob.GetBufferPointer(), byteCode, 0, byteCode.Length);
        var pixelShader = new PixelShader();
        using var stream = new MemoryStream(byteCode, writable: false);
        pixelShader.SetStreamSource(stream);
        return pixelShader;
    }
    finally
    {
        Release(errorBlob);
        Release(shaderBlob);
    }
}
```

`D3DCompile` 返回的是 HRESULT，负数表示失败。失败时优先从 `errorBlob` 里读出错误文本，能直接看到 HLSL 的编译报错，这对调试着色器非常重要，不然只能看到一个没有信息的 HRESULT

编译成功之后，把 `ID3DBlob` 里的字节码复制到托管数组，包成 `MemoryStream` 交给 `PixelShader.SetStreamSource`。这里的 `checked((int)shaderBlob.GetBufferSize())` 是为了防止在 32 位环境下指针大小转换溢出

`finally` 块里释放两个 COM 对象很关键。`ID3DBlob` 是 COM 对象，不释放会一直占用内存，而 `Release` 方法做了类型检查再调用 `Marshal.ReleaseComObject`：

```csharp
private static void Release(object? value)
{
    if (value is not null && Marshal.IsComObject(value))
    {
        Marshal.ReleaseComObject(value);
    }
}
```

## 封装 ShaderEffect

像素着色器编译好之后，需要一个 WPF 的 `ShaderEffect` 子类把它包装起来，才能在 XAML 里使用。`PerspectiveEffect` 类就是干这个的

静态字段保存编译结果，整个程序只需要编译一次：

```csharp
private static readonly PixelShader Shader = RuntimePixelShaderCompiler.Compile();
```

接着注册依赖属性。`InputProperty` 用 `RegisterPixelShaderSamplerProperty` 注册，这是采样器属性，对应 HLSL 里的 `s0` 寄存器：

```csharp
public static readonly DependencyProperty InputProperty = RegisterPixelShaderSamplerProperty(
    nameof(Input), typeof(PerspectiveEffect), 0);
```

矩阵的三行各用一个 `Point4D` 类型的依赖属性，通过 `PixelShaderConstantCallback` 绑定到常量寄存器：

```csharp
public static readonly DependencyProperty MatrixRow0Property = DependencyProperty.Register(
    nameof(MatrixRow0), typeof(Point4D), typeof(PerspectiveEffect),
    new UIPropertyMetadata(new Point4D(1, 0, 0, 0), PixelShaderConstantCallback(0)));

public static readonly DependencyProperty MatrixRow1Property = DependencyProperty.Register(
    nameof(MatrixRow1), typeof(Point4D), typeof(PerspectiveEffect),
    new UIPropertyMetadata(new Point4D(0, 1, 0, 0), PixelShaderConstantCallback(1)));

public static readonly DependencyProperty MatrixRow2Property = DependencyProperty.Register(
    nameof(MatrixRow2), typeof(Point4D), typeof(PerspectiveEffect),
    new UIPropertyMetadata(new Point4D(0, 0, 1, 0), PixelShaderConstantCallback(2)));
```

为什么用 `Point4D` 而不是直接传 3x3 矩阵？因为 GPU 常量寄存器是按 `float4` 对齐的，`PixelShaderConstantCallback` 只能传 `float4`，所以把矩阵的每一行放进一个 `Point4D`，第四分量闲置为 0。默认值是单位矩阵，这样即使还没设置矩阵，效果也不会把图片弄丢

构造函数里把编译好的着色器赋给 `PixelShader`，并刷新所有属性的值，让默认值生效：

```csharp
public PerspectiveEffect()
{
    PixelShader = Shader;
    UpdateShaderValue(InputProperty);
    UpdateShaderValue(MatrixRow0Property);
    UpdateShaderValue(MatrixRow1Property);
    UpdateShaderValue(MatrixRow2Property);
}
```

最后提供 `SetInverseMatrix` 方法，把 3x3 矩阵拆成三行填进 `Point4D`：

```csharp
public void SetInverseMatrix(double[,] matrix)
{
    ArgumentNullException.ThrowIfNull(matrix);
    MatrixRow0 = new Point4D(matrix[0, 0], matrix[0, 1], matrix[0, 2], 0);
    MatrixRow1 = new Point4D(matrix[1, 0], matrix[1, 1], matrix[1, 2], 0);
    MatrixRow2 = new Point4D(matrix[2, 0], matrix[2, 1], matrix[2, 2], 0);
}
```

注意传入的是逆矩阵，因为着色器需要的是"输出像素到源纹理"的映射。赋值依赖属性会自动触发 `PixelShaderConstantCallback`，把数据传到 GPU 的常量寄存器，无需手动刷新

## 界面布局

界面布局在 MainWindow.xaml 里。窗口采用深色主题，核心区域是一个固定逻辑尺寸 900x560 的 Viewport，外层套 Viewbox 让它随窗口缩放：

```xml
<Viewbox Stretch="Uniform">
    <Grid x:Name="Viewport" Width="900" Height="560" ClipToBounds="True">
        ...
    </Grid>
</Viewbox>
```

`ClipToBounds="True"` 防止变换后的图片画出边界。Viewport 内部依次堆叠三层内容：最底层是棋盘格背景，中间是图片，最上层是交互层

棋盘格背景用 DrawingBrush 平铺实现，模仿设计软件里的透明棋盘，用来衬托四边形外的透明区域：

```xml
<Border>
    <Border.Background>
        <DrawingBrush TileMode="Tile" Viewport="0,0,24,24" ViewportUnits="Absolute">
            <DrawingBrush.Drawing>
                <DrawingGroup>
                    <GeometryDrawing Brush="#242424" Geometry="M0,0 H24 V24 H0 Z" />
                    <GeometryDrawing Brush="#333333" Geometry="M0,0 H12 V12 H0 Z M12,12 H24 V24 H12 Z" />
                </DrawingGroup>
            </DrawingBrush.Drawing>
        </DrawingBrush>
    </Border.Background>
</Border>
```

24x24 的方块，左边画深色，右边画浅色，形成棋盘格

图片层直接显示一张示例图，`Stretch="Fill"` 让图片填满整个 Viewport，这样单应变换的输入就是整张图：

```xml
<Image x:Name="PreviewImage"
       Source="/Image.png"
       Stretch="Fill"
       RenderOptions.BitmapScalingMode="HighQuality" />
```

最上层的 Canvas 放四边形轮廓和四个拖拽手柄。轮廓是虚线 Polygon，手柄是四个 Thumb，用 Tag 标记角点索引：

```xml
<Canvas x:Name="AdornerCanvas" Background="Transparent">
    <Polygon x:Name="Outline"
             Fill="Transparent"
             Stroke="#F9AB00"
             StrokeThickness="2"
             StrokeDashArray="5,3"
             IsHitTestVisible="False" />
    <Thumb x:Name="TopLeftThumb" Tag="0" Style="{StaticResource CornerThumbStyle}"
           DragDelta="CornerThumb_OnDragDelta" />
    <Thumb x:Name="TopRightThumb" Tag="1" Style="{StaticResource CornerThumbStyle}"
           DragDelta="CornerThumb_OnDragDelta" />
    <Thumb x:Name="BottomRightThumb" Tag="2" Style="{StaticResource CornerThumbStyle}"
           DragDelta="CornerThumb_OnDragDelta" />
    <Thumb x:Name="BottomLeftThumb" Tag="3" Style="{StaticResource CornerThumbStyle}"
           DragDelta="CornerThumb_OnDragDelta" />
</Canvas>
```

Thumb 是 WPF 自带的可拖拽控件，非常适合做这种手柄。它的外观通过 `CornerThumbStyle` 定制成黄色圆环加白色描边，Cursor 设为 `SizeAll` 提示可拖拽

## 拖拽交互

交互逻辑在 MainWindow.xaml.cs 里。所有 Thumb 共用同一个 `DragDelta` 事件处理函数，通过 `Tag` 区分是哪个角点：

```csharp
private void CornerThumb_OnDragDelta(object sender, DragDeltaEventArgs e)
{
    var thumb = (Thumb)sender;
    var index = int.Parse((string)thumb.Tag);
    var current = _corners[index];
    var candidate = new Point(
        Math.Clamp(current.X + (e.HorizontalChange / Viewport.Width), 0, 1),
        Math.Clamp(current.Y + (e.VerticalChange / Viewport.Height), 0, 1));

    var previous = _corners[index];
    _corners[index] = candidate;

    if (!Homography.IsConvexQuadrilateral(_corners))
    {
        _corners[index] = previous;
        return;
    }

    UpdateVisuals();
}
```

`DragDelta` 给的是本次拖动的像素位移，把它除以 Viewport 的宽高，就换算成了归一化坐标的增量，和 `_corners` 的坐标系保持一致。`Math.Clamp` 把角点限制在 0 到 1 范围内，防止拖出图片区域

这里有一个很妙的细节：先更新角点，再校验凸四边形，如果校验失败就把角点回滚到拖动前的值。也就是说，允许用户把角点拖到"边缘极限"的合法位置，但一旦越过凸四边形的底线，就停留在最后合法状态，不会出现无法恢复的畸形变换

最后调用 `UpdateVisuals` 刷新界面。这个方法把"计算矩阵"和"绘制 UI"串联起来：

```csharp
private void UpdateVisuals()
{
    var inverse = Homography.CreateQuadrilateralToUnitSquare(_corners);
    _effect.SetInverseMatrix(inverse);

    var points = _corners
        .Select(point => new Point(point.X * Viewport.Width, point.Y * Viewport.Height))
        .ToArray();

    Outline.Points = new PointCollection(points);
    PositionThumb(TopLeftThumb, points[0]);
    PositionThumb(TopRightThumb, points[1]);
    PositionThumb(BottomRightThumb, points[2]);
    PositionThumb(BottomLeftThumb, points[3]);
}
```

先用当前角点算出逆矩阵设置到 Effect，这一行就完成了"数据到 GPU"的传递，着色器马上会用新矩阵重新渲染。然后把归一化坐标乘回 Viewport 的宽高，得到像素坐标，用于更新虚线轮廓和四个手柄的位置。手柄定位时减去 `ThumbRadius`，让手柄圆心对准角点：

```csharp
private static void PositionThumb(Thumb thumb, Point point)
{
    Canvas.SetLeft(thumb, point.X - ThumbRadius);
    Canvas.SetTop(thumb, point.Y - ThumbRadius);
}
```

构造函数里把 Effect 挂到图片上，并做一次初始绘制：

```csharp
public MainWindow()
{
    InitializeComponent();
    PreviewImage.Effect = _effect;
    UpdateVisuals();
}
```

到这里整个 Demo 就闭环了：拖动任意手柄，角点数组变化，矩阵重算，着色器重绘，图片实时变形

## 总结

这个 Demo 麻雀虽小五脏俱全，串起了三条技术线：单应矩阵的数学计算、HLSL 像素着色器、WPF ShaderEffect 与 COM 互操作。其中最容易踩坑的是运行时编译那一段，`ID3DBlob` 的 COM 接口声明、`[PreserveSig]` 特性、COM 对象释放，任何一个疏漏都会导致编译失败或内存泄漏，建议直接复用本文的封装

如果想要完全了解代码的每个细节，本文末尾有获取全部代码的方法

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/f3a5a4e0c4572d7b4e10ce7dc3cb5496714a9629/WPFDemo/HomographyShaderEffectDemo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f3a5a4e0c4572d7b4e10ce7dc3cb5496714a9629/WPFDemo/HomographyShaderEffectDemo) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f3a5a4e0c4572d7b4e10ce7dc3cb5496714a9629
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f3a5a4e0c4572d7b4e10ce7dc3cb5496714a9629
```

获取代码之后，进入 WPFDemo/HomographyShaderEffectDemo 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )