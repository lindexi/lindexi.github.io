---
title: WPF 性能测试
description: 本文收藏我给 WPF 做的性能测试。在你开始认为 WPF 的性能存在问题的时候，不妨来这篇博客里找找看我做过的测试。我记录的测试都是比较纯净的测试项目，没有业务逻辑的干扰，写法也正常，可以更加真实反映 WPF 的性能，减少因为奇怪的业务逻辑以及逗比的写法的影响
tags: WPF
category: 
---

<!-- CreateTime:2023/3/9 9:14:41 -->

<!-- 发布 -->
<!-- 博客 -->

## 资源字典

### 大量 Geometry 资源对启动的影响

在资源字典里面存放了 5k 个 Geometry 对象，资源字典加入到 App.Xaml 的资源字典里面，意味着启动时就会加载这些资源。根据 WPF 对资源对象创建的定义，可以了解到，在 WPF 里面不会立刻创建资源对象，只有在资源对象首次被使用时才会被创建。也就是说加入到 App.Xaml 的资源字典的 5k 个 Geometry 对象将只会被记录到 App 的资源字典里面，但没有实际创建出来

实际测试性能大概是在我电脑上加载只需 50 毫秒左右

以上测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/1d3883ac7feba5dd7752e1edccd33f943c02f7f9/JojeryiheenelNearfinelwhea) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1d3883ac7feba5dd7752e1edccd33f943c02f7f9/JojeryiheenelNearfinelwhea) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个名为 JojeryiheenelNearfinelwhea 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1d3883ac7feba5dd7752e1edccd33f943c02f7f9
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1d3883ac7feba5dd7752e1edccd33f943c02f7f9
```

获取代码之后，进入 JojeryiheenelNearfinelwhea 文件夹

### 大量资源图片对启动的影响

我创建了 100 张图片，将这些图片作为资源的存在，接着写一个 资源字典 引用这 100 张图片。这 100 张图片都属于不同的图片，最后构建出来的 DLL 文件大概有 300 MB 这么大

![](http://cdn.lindexi.site/lindexi%2F20234221458573021.jpg)

将资源字典同样在 App.xaml 里引用加入，测量 App 的 InitializeComponent 时间发现近乎没有受到图片数量的影响。在我的设置上 Debug 模式下仅不到百毫秒即可完成，即使我是放在机械硬盘上运行的

我编写的测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/9c660568dfadef19c3393a42ca0925b9e72cd749/WhirawahereRallcobaiwe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/9c660568dfadef19c3393a42ca0925b9e72cd749/WhirawahereRallcobaiwe) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个名为 WhirawahereRallcobaiwe 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 9c660568dfadef19c3393a42ca0925b9e72cd749
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 9c660568dfadef19c3393a42ca0925b9e72cd749
```

获取代码之后，进入 WhirawahereRallcobaiwe 文件夹。里面包含一个生成测试图片和测试代码的项目和一个用来测试启动性能的 WPF 项目

以下是生成测试图片的代码

```csharp
                    WriteableBitmap writeableBitmap = new WriteableBitmap(1024, 1024, 96, 96, PixelFormats.Pbgra32, null);

                    writeableBitmap.Lock();
                    unsafe
                    {
                        var length = writeableBitmap.PixelWidth * writeableBitmap.PixelHeight *
                           writeableBitmap.Format.BitsPerPixel / 8;
                        var backBuffer = (byte*) writeableBitmap.BackBuffer;
                        for (int i = 0; i + 4 < length; i = i + 4)
                        {
                            Span<byte> span = new Span<byte>(backBuffer, length);
                            span = span.Slice(i, 4);
                            Random.Shared.NextBytes(span);
                        }
                    }

                    writeableBitmap.Unlock();

                    PngBitmapEncoder encoder = new PngBitmapEncoder();
                    encoder.Frames.Add(BitmapFrame.Create(writeableBitmap));

                    var file = $"{fileName}.png";

                    using var fileStream = File.OpenWrite(file);
                    encoder.Save(fileStream);
```

可以看到是采用随机的像素的方式生成的图片，如此即可让每个图片保证是不同的。从以上测试项目可以了解到，假定在启动过程中 WPF 框架是做了图片加载和解析的工作的，那 WPF 的启动时间绝对不可能有这么快，我猜测在许多年内都无法在机械盘上不到百毫秒内完成 100 张 1024x1024 的不同图片的解码。换句话说就是 WPF 的资源字典里面定义的引用确实是延迟加载的，在使用到的时候才真正创建对象，这符合我阅读 WPF 源代码所了解的


## 一千个半透明矩形做动画

[WPF 动画性能测试应用 一千个半透明矩形做动画](https://blog.lindexi.com/post/WPF-%E5%8A%A8%E7%94%BB%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E5%BA%94%E7%94%A8-%E4%B8%80%E5%8D%83%E4%B8%AA%E5%8D%8A%E9%80%8F%E6%98%8E%E7%9F%A9%E5%BD%A2%E5%81%9A%E5%8A%A8%E7%94%BB.html )

## 画10万个矩形


测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/4983492acb47c040ecb80b7417f7cf364d1e3e19/NarlearcefearNuyikallair) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/4983492acb47c040ecb80b7417f7cf364d1e3e19/NarlearcefearNuyikallair) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个名为 NarlearcefearNuyikallair 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 4983492acb47c040ecb80b7417f7cf364d1e3e19
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 4983492acb47c040ecb80b7417f7cf364d1e3e19
```

获取代码之后，进入 NarlearcefearNuyikallair 文件夹

带刷新率的绘制一万矩形版本的测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/9e4fb7c067598fb133c9c1ade73b45e27f4cea67/FekemreakairlayHijehereci) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/9e4fb7c067598fb133c9c1ade73b45e27f4cea67/FekemreakairlayHijehereci) 上

## 画千线

测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/83621f444c741eb424f1a52fbac503d04b7608e0/WPFDemo/CebemwarjawkeJaihokahaiqere) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/83621f444c741eb424f1a52fbac503d04b7608e0/WPFDemo/CebemwarjawkeJaihokahaiqere) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 83621f444c741eb424f1a52fbac503d04b7608e0
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 83621f444c741eb424f1a52fbac503d04b7608e0
```

获取代码之后，进入 WPFDemo/CebemwarjawkeJaihokahaiqere 文件夹，即可获取到源代码

## 视频播放性能

[WPF 模拟 WPFMediaKit 的 D3D 配置用来测试4k性能](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F-WPFMediaKit-%E7%9A%84-D3D-%E9%85%8D%E7%BD%AE%E7%94%A8%E6%9D%A5%E6%B5%8B%E8%AF%954k%E6%80%A7%E8%83%BD.html )

## WriteableBitmap 的拷贝性能

写 WriteableBitmap 的时候，调用 WritePixels 方法，进行一次 CPU->CPU 的内存拷贝。在 WPF 框架层，还会从 WriteableBitmap 执行一次 CPU->CPU 的内存拷贝，让数据到达渲染线程。最终在渲染阶段，需要来一次 CPU->GPU 的拷贝，从而在界面渲染出来。详细请参阅 [dotnet 读 WPF 源代码笔记 WriteableBitmap 的渲染和更新是如何实现](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-WriteableBitmap-%E7%9A%84%E6%B8%B2%E6%9F%93%E5%92%8C%E6%9B%B4%E6%96%B0%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0.html )

本次测试内容就是一整个界面都是 Image 控件，这个 Image 控件带一个和窗口一样大（去掉标题栏哈）的 WriteableBitmap 对象，每次渲染都向其赋值新的预设内存数组。从而进行测试 WriteableBitmap 的拷贝和渲染性能

核心代码如下，以下代码不计性能代价，甚至可以完全跑满 GPU 资源。预计在 4K 的核心卡点在于内存带宽和 GPU 资源上

```csharp
    public MainWindow()
    {
        InitializeComponent();
        Loaded += MainWindow_Loaded;
        _bufferEnumerator = GetBuffer().GetEnumerator();
    }

    private void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        Image.Width = ActualWidth;
        Image.Height = ActualHeight;

        var width = (int)ActualWidth;
        var height = (int)ActualHeight;

        var writeableBitmap =
            new WriteableBitmap(width, (int)ActualHeight, 96, 96, PixelFormats.Bgra32, null);
        _writeableBitmap = writeableBitmap;
        Image.Source = writeableBitmap;

        for (int i = 0; i < 60; i++)
        {
            var buffer = new byte[4 * width * height];
            PixelBufferList.Add(buffer);
            FillBuffer(buffer, i);
        }

        Redraw();
    }

    private void FillBuffer(byte[] buffer, int index)
    {
        for (var i = 0; i < buffer.Length; i += 4)
        {
            buffer[i] = (byte)(10 * index);
            buffer[i + 1] = 0;
            buffer[i + 2] = 0xF1;
            buffer[i + 3] = 0xFF;

            Random.Shared.NextBytes(buffer.AsSpan(i, 3));

            buffer[i + 3] = 0xFF;
        }

        Debug.Assert(_writeableBitmap != null);
        var width = _writeableBitmap.PixelWidth;
        var height = _writeableBitmap.PixelHeight;

        for (int heightIndex = 100; heightIndex < Math.Min(height, 300); heightIndex++)
        {
            for (int widthIndex = 50; widthIndex < Math.Min(width, (index + 1) * (width / 60)); widthIndex++)
            {
                var span = buffer.AsSpan(heightIndex * width * 4 + widthIndex * 4, 3);
                span[0] = 0xFF;
                span[1] = 0x00;
                span[2] = 0x00;
            }
        }
    }

    private List<byte[]> PixelBufferList { get; } = [];
    private WriteableBitmap? _writeableBitmap;

    private readonly IEnumerator<byte[]> _bufferEnumerator;

    private IEnumerable<byte[]> GetBuffer()
    {
        while (true)
        {
            foreach (var buffer in PixelBufferList)
            {
                yield return buffer;
            }
        }
    }

    protected override void OnRender(DrawingContext drawingContext)
    {
        Dispatcher.InvokeAsync(Redraw, DispatcherPriority.Input);
    }

    private void Redraw()
    {
        Debug.Assert(_writeableBitmap != null);

        _bufferEnumerator.MoveNext();
        var buffer = _bufferEnumerator.Current;
        _writeableBitmap.Lock();
        _writeableBitmap.WritePixels(new Int32Rect(0, 0, _writeableBitmap.PixelWidth, _writeableBitmap.PixelHeight),
            buffer, 4 * _writeableBitmap.PixelWidth, 0);
        _writeableBitmap.Unlock();

        InvalidateVisual();
    }
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b14c7cca5ad1fae96cbb3a60ca9977d872fb8941/WPFDemo/NebalnefaichallkereQalayhearchal) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/b14c7cca5ad1fae96cbb3a60ca9977d872fb8941/WPFDemo/NebalnefaichallkereQalayhearchal) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b14c7cca5ad1fae96cbb3a60ca9977d872fb8941
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b14c7cca5ad1fae96cbb3a60ca9977d872fb8941
```

获取代码之后，进入 WPFDemo/NebalnefaichallkereQalayhearchal 文件夹，即可获取到源代码。欢迎拉取代码进行测试

如对框架底层的 WriteableBitmap 更新渲染感兴趣，请参阅 [dotnet 读 WPF 源代码笔记 WriteableBitmap 的渲染和更新是如何实现](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-WriteableBitmap-%E7%9A%84%E6%B8%B2%E6%9F%93%E5%92%8C%E6%9B%B4%E6%96%B0%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0.html ) 和 [WPF 从 WriteableBitmap 里获取到渲染线程使用的 IWICBitmap 对象](https://blog.lindexi.com/post/WPF-%E4%BB%8E-WriteableBitmap-%E9%87%8C%E8%8E%B7%E5%8F%96%E5%88%B0%E6%B8%B2%E6%9F%93%E7%BA%BF%E7%A8%8B%E4%BD%BF%E7%94%A8%E7%9A%84-IWICBitmap-%E5%AF%B9%E8%B1%A1.html )
<!-- [WPF 从 WriteableBitmap 里获取到渲染线程使用的 IWICBitmap 对象 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18843888 ) -->
