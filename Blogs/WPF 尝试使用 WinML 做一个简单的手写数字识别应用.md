---
title: WPF 尝试使用 WinML 做一个简单的手写数字识别应用
description: 最近我看了微软的 AI 训练营之后，似乎有点了解 Windows Machine Learning 和 DirectML 的概念，于是我尝试实践一下，用 WPF 写一个简单的触摸手写输入的画板，再使用大佬训练好的 mnist.onnx 模型，对接 WinML 实现一个简单的手写数字识别应用
tags: WPF
category: 
---

<!-- CreateTime:2023/12/9 17:27:59 -->
<!-- 发布 -->
<!-- 博客 -->

本文属于 WinML 的入门级博客，我将尝试一步步告诉大家，如何对接 Windows AI 里的 Windows Machine Learning（WinML）使用已训练好的 onnx 模型

本文的一些概念写于 2023年12月，感觉微软会经常改概念，要是本文编写时间距离你当前阅读时间过远，可能本文有些概念已经不正确了

本文将要介绍的 WinML 是 Windows AI 集里面的一个功能点，此功能叫 Windows Machine Learning 意味着这是和系统绑定的功能，想要使用此功能，要求使用 Win10 1809 或以上的系统版本

根据微软[官方文档](https://learn.microsoft.com/en-us/windows/ai/windows-ml/get-started)可以知道 WinML 底层里有依赖 DirectML 组件，从架构层级上看大概的关系图如下

<!-- ![](image/WPF 尝试使用 WinML 做一个简单的手写数字识别应用/WPF 尝试使用 WinML 做一个简单的手写数字识别应用0.png) -->
![](https://img2024.cnblogs.com/blog/1080237/202509/1080237-20250917072949546-3396624.png)

从设计上 DirectML 是底层的，通过高可控实现高性能，但高可控带来的副作用是使用麻烦，不适合应用程序直接使用，更多的是需要在此之前封装一层框架方便应用程序对接。而 WinML 正是这样的一层封装，通过 WinML 提供的较友好的 API 可以方便应用程序实现大部分业务功能

使用 WinML 提供的上层人类友好的 API 不仅可以间接使用到 DirectML 提供的对 GPU 或其他加速设备的硬件加速，还可以在设备硬件缺失或不允许的情况下自动调度到 CPU 上运行

接下来我将演示的代码是采用 WinRT 的方式调用 WinML 层。先新建一个空 WPF 应用，在本文末尾我放上了本文用到的全部代码的下载方法。再配置引用 WindowsAppSDK 库，通过 WindowsAppSDK 的方式使用到 WinRT 组件。快捷的安装 WindowsAppSDK 的方法是编辑 csproj 项目文件，编辑之后的项目文件内容大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows10.0.19041</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
    <PlatformTarget>x86</PlatformTarget>
    <RuntimeIdentifiers>win10-x86;win10-x64</RuntimeIdentifiers>
    <TargetPlatformMinVersion>10.0.17763.0</TargetPlatformMinVersion>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.3.230331000" />
  </ItemGroup>
  <ItemGroup>
    <None Update="mnist.onnx"> <!-- 这是我下载的模型文件，设置拷贝输出方便加载模型 -->
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>
</Project>
```

以上代码的最低版本要求设置为 10.0.17763.0 就是对应 1809 版本。而在 TargetFramework 里面的 10.0.19041 是对应 2004 版本。版本之间的关系图如下，详细请看 [Windows 10 version history - Wikipedia](https://en.wikipedia.org/wiki/Windows_10_version_history )

<!-- ![](image/WPF 尝试使用 WinML 做一个简单的手写数字识别应用/WPF 尝试使用 WinML 做一个简单的手写数字识别应用1.png) -->
![](https://img2024.cnblogs.com/blog/1080237/202509/1080237-20250917072950298-1755431630.png)

接下来下载大佬训练好的 mnist.onnx 模型文件，下载地址是 [https://github.com/microsoft/Windows-Machine-Learning/raw/master/Samples/MNIST/UWP/cs/Assets/mnist.onnx](https://github.com/microsoft/Windows-Machine-Learning/raw/master/Samples/MNIST/UWP/cs/Assets/mnist.onnx)

我在本文末尾的代码仓库里面也包含了此文件，大家也可以从 gitee 国内源拉取

为了方便做一个演示应用，接下来添加一个简单的界面代码，也就是放两个按钮，一个用来做识别，一个用来做清理。再放上一个 InkCanvas 控件用来写内容，且由于接下来的 mnist.onnx 模型走的是图像识别的方式，为了提高识别率，还需要让写出来的笔迹粗一些。最后添加一个 TextBlock 用来显示识别的输出

```xml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition></RowDefinition>
            <RowDefinition Height="Auto"></RowDefinition>
            <RowDefinition Height="Auto"></RowDefinition>
        </Grid.RowDefinitions>
        <InkCanvas x:Name="InkCanvas">
            <InkCanvas.DefaultDrawingAttributes>
                <DrawingAttributes Width="22" Height="22"></DrawingAttributes>
            </InkCanvas.DefaultDrawingAttributes>
        </InkCanvas>
        <Grid Grid.Row="1">
            <Button x:Name="RecognizeButton" Margin="10,10,10,10" HorizontalAlignment="Left" Click="RecognizeButton_OnClick">识别</Button>
            <Button x:Name="ClearButton" Margin="10,10,10,10" HorizontalAlignment="Right" Click="ClearButton_OnClick">清空</Button>
        </Grid>
        <TextBlock x:Name="TextBlock" Grid.Row="2" Margin="10,10,10,10"></TextBlock>
    </Grid>
```

切换到后台代码，在 MainWindow 的构造函数里面，先准备加载模型。加载模型的时候，可以设置 LearningModelDevice 的 LearningModelDeviceKind 类型，决定采用什么样的设备去执行模型，可选参数如下

- Cpu：使用 CPU 执行模型

- Default：默认，让系统自己选择使用设备，也就是比较推荐的方法

- DirectX ： 使用 GPU 或其他的 DirectX 设备执行模型

- DirectXHighPerformance ： 使用系统里面定制的属于高性能的设备的 DirectX 设备执行模型。比如独显

- DirectXMinPower ： 使用系统里面定制的属于低功耗的设备的 DirectX 设备执行模型

从文件加载模型，代码如下

```csharp
        LearningModel learningModel = LearningModel.LoadFromFilePath("mnist.onnx");
```

接下来是创建 LearningModelSession 和 LearningModelBinding 对象，代码如下

```csharp
    public MainWindow()
    {
        InitializeComponent();

        LearningModel learningModel = LearningModel.LoadFromFilePath("mnist.onnx");
        var deviceToRunOn = new LearningModelDevice(LearningModelDeviceKind.DirectXHighPerformance);
        var learningModelSession = new LearningModelSession(learningModel, deviceToRunOn);
        var learningModelBinding = new LearningModelBinding(learningModelSession);

        LearningModel = learningModel;
        LearningModelSession = learningModelSession;
        LearningModelBinding = learningModelBinding;
    }

    public LearningModel LearningModel { get; set; }

    public LearningModelSession LearningModelSession { get; set; }

    public LearningModelBinding LearningModelBinding { get; set; }
```

在点击识别按钮，就需要将 InkCanvas 内容转换为 Windows.Media.VideoFrame 对象，用于传入到模型里面进行识别

在 WPF 里面对控件进行截图，可以使用 RenderTargetBitmap 进行截图，代码十分简单

```csharp
        var width = (int) InkCanvas.ActualWidth;
        var height = (int) InkCanvas.ActualHeight;

        var bitmapSource = new RenderTargetBitmap(width, height, 96, 96, PixelFormats.Pbgra32);
        bitmapSource.Render(InkCanvas);
```

为了构建 Windows.Media.VideoFrame 对象，咱需要中间的 SoftwareBitmap 对象作为辅助。创建 SoftwareBitmap 可以从像素数组进行创建，获取 RenderTargetBitmap 的像素数组的方法可以是先开辟一个缓存空间，让 RenderTargetBitmap 将像素数组写入到缓存空间里面。在 WPF 里面，渲染不是实时发生的，换句话说是如果你不从 RenderTargetBitmap 里面让其输出像素数组，实际上内部是没有干多少活的

```csharp
        var stride = bitmapSource.PixelWidth * bitmapSource.Format.BitsPerPixel / 8;
        var length = stride * bitmapSource.PixelHeight;
        var byteArray = new byte[length];
        bitmapSource.CopyPixels(byteArray, stride, 0);
```

输出的代码需要记住使用的像素格式与 byte 数组长度之间的关系。以上的 stride 可以认为是 2D 图片里面的每一行使用的 byte 数量

拿到像素数组之后，转换为 Windows.Storage.Streams.IBuffer 对象，用于创建 SoftwareBitmap 对象

```csharp
        IBuffer buffer = byteArray.AsBuffer();
```

通过像素数组创建 SoftwareBitmap 对象需要指定传入的像素数组格式，在 WinRT 里面与 WPF 的 PixelFormats.Pbgra32 对应的是 BitmapPixelFormat.Bgra8 格式。在 WPF 的 PixelFormats.Pbgra32 表示的是使用一个 32 位的空间表示一个像素，像素顺序是 B 蓝色 G 绿色 R 红色。在 WinRT 的 BitmapPixelFormat.Bgra8 表示使用 8 个位表示 B 蓝色，使用 8 个位表示 G 绿色，使用 8 个位表示 R 红色，总共也是 32 位，和 WPF 的 Pbgra32 其实是完全相同的，只是说法不同而已

```csharp
        var softwareBitmap = SoftwareBitmap.CreateCopyFromBuffer(buffer, BitmapPixelFormat.Bgra8, bitmapSource.PixelWidth, bitmapSource.PixelHeight);
```

拿到 SoftwareBitmap 之后即可创建 VideoFrame 对象，代码如下

```csharp
        VideoFrame inputImage = VideoFrame.CreateWithSoftwareBitmap(softwareBitmap);
```

将 VideoFrame 作为模型的输入参数，代码如下

```csharp
        var imageFeatureValue = ImageFeatureValue.CreateFromVideoFrame(inputImage);

        LearningModelBinding.Bind("Input3", imageFeatureValue);
```

以上为什么使用 `"Input3"` 作为输入的绑定源？这是因为大佬在训练的 mnist.onnx 就这样写了，详细可以在 LearningModel.LoadFromFilePath 之后，通过 LearningModel 的 InputFeatures 属性看到输入的要求。同理可以从 OutputFeatures 属性获取模型输出的描述

完成绑定输入之后，即可通过 LearningModelSession 开始执行模型，和执行完成之后获取其模型结果的代码如下

```csharp
        var result = await LearningModelSession.EvaluateAsync(LearningModelBinding, "0");

        var resultOutput = result.Outputs["Plus214_Output_0"] as TensorFloat;
        var vectorView = resultOutput?.GetAsVectorView();
        if (vectorView != null)
        {
            var maxValue = 0f;
            var maxIndex = -1;
            // 10 个数字，每个数字
            for (var number = 0; number < vectorView.Count; number++)
            {
                Debug.WriteLine($"{number} {vectorView[number]}");

                if (vectorView[number] > maxValue)
                {
                    maxValue = vectorView[number];
                    maxIndex = number;
                }
            }

            if (maxIndex == -1)
            {
                TextBlock.Text = $"识别失败";
            }
            else
            {
                TextBlock.Text = $"识别数字：{maxIndex} 识别率：{maxValue}";
            }
        }
```

这个模型的输出很有趣，是输出一个包含 10 个元素的 float 数组，这个数组上的每个元素代表着对应的数字的识别率或者说可信度是多少。因此找到可信度最高的元素对应的下标就是这个模型识别到的数字

比如说 `Debug.WriteLine($"{number} {vectorView[number]}")` 输出的 `vectorView` 内容如下

```
0 193.39949
1 76.210556
2 -537.7121
3 -514.7019
4 299.47296
5 559.53064
6 402.9799
7 -929.9102
8 -272.99913
9 -1143.2314
```

也就是下标为 5 的数值 559.53064 最大，证明模型识别出来的数字应该就是 5 这个数字

实现识别按钮的代码如下

```csharp
    private async void RecognizeButton_OnClick(object sender, RoutedEventArgs e)
    {
        var width = (int) InkCanvas.ActualWidth;
        var height = (int) InkCanvas.ActualHeight;

        var bitmapSource = new RenderTargetBitmap(width, height, 96, 96, PixelFormats.Pbgra32);
        bitmapSource.Render(InkCanvas);

        var length = bitmapSource.PixelWidth * bitmapSource.PixelHeight * bitmapSource.Format.BitsPerPixel / 8;
        var byteArray = new byte[length];
        var stride = bitmapSource.PixelWidth * bitmapSource.Format.BitsPerPixel / 8;
        bitmapSource.CopyPixels(byteArray, stride, 0);

        IBuffer buffer = byteArray.AsBuffer();

        var softwareBitmap = SoftwareBitmap.CreateCopyFromBuffer(buffer, BitmapPixelFormat.Bgra8, bitmapSource.PixelWidth, bitmapSource.PixelHeight);
       
        VideoFrame inputImage = VideoFrame.CreateWithSoftwareBitmap(softwareBitmap);

        var imageFeatureValue = ImageFeatureValue.CreateFromVideoFrame(inputImage);

        LearningModelBinding.Bind("Input3", imageFeatureValue);

        var result = await LearningModelSession.EvaluateAsync(LearningModelBinding, "0");

        var resultOutput = result.Outputs["Plus214_Output_0"] as TensorFloat;
        var vectorView = resultOutput?.GetAsVectorView();
        if (vectorView != null)
        {
            var maxValue = 0f;
            var maxIndex = -1;
            // 10 个数字，每个数字
            for (var number = 0; number < vectorView.Count; number++)
            {
                Debug.WriteLine($"{number} {vectorView[number]}");

                if (vectorView[number] > maxValue)
                {
                    maxValue = vectorView[number];
                    maxIndex = number;
                }
            }

            if (maxIndex == -1)
            {
                TextBlock.Text = $"识别失败";
            }
            else
            {
                TextBlock.Text = $"识别数字：{maxIndex} 识别率：{maxValue}";
            }
        }
    }
```

通过上文的介绍，大家可以了解如何在 WPF 应用里面，通过 WinRT 的方式对接 WinML 层。以及如何加载大佬们训练好的 onnx 模型，和如何执行模型

如果对此演示项目的代码感兴趣，可以通过下面方式获取到本文的所有代码

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/29df54af3d2471c002303eb412372b844b0708bc/BenukalliwayaChayjanehall) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/29df54af3d2471c002303eb412372b844b0708bc/BenukalliwayaChayjanehall) 欢迎访问

可以通过如下方式获取本文以上的源代码，先创建一个名为 BenukalliwayaChayjanehall 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 29df54af3d2471c002303eb412372b844b0708bc
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 29df54af3d2471c002303eb412372b844b0708bc
```

获取代码之后，进入 BenukalliwayaChayjanehall 文件夹
