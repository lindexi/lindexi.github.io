# win10 uwp win2d

本文主要翻译，可能带有一定的主观性和局限性，说的东西可能不对或者不符合每个人的预期。如果觉得我有讲的不对的，就多多包含，或者直接关掉这篇文章，但是请勿生气或者发怒吐槽，可以在我博客评论 http://blog.csdn.net/lindexi_gd
<!--more-->

<div id="toc"></div>
<!-- csdn -->

## 介绍

Win2d是一个很简单使用的底层图形Windows Runtime API，可以使用硬件加速，主要是GPU的强大计算。他可以使用C#或C++写应用商店应用，包括UWP或windows 8.1手机或电脑。他利用强大的Direct2D，无缝集合windows的Xaml，可以使用强大的渲染得到漂亮界面。

使用他可以将界面交给GPU，让CPU集中计算我们的算法

我们可以通过Nuget来得到win2d，Nuget的windows10版win2d:[http://www.nuget.org/packages/Win2D.uwp](http://www.nuget.org/packages/Win2D.uwp)，Nuget的windows 8.1版win2d:http://www.nuget.org/packages/Win2D.win81

如何使用可以参见微软示例http://github.com/Microsoft/Win2D-samples

在下面我们会说如何快速使用。

一些链接：
如果找到bug可以通过 http://github.com/Microsoft/Win2D/issues
团队博客：http://blogs.msdn.com/b/win2d


## 特性

- 位映像图形
 
 - 加载、保存、渲染图形
  
 - 纹理渲染
 
 - 蒙版
 
 - 快速处理大量图片
 
 - 分块压缩图像
 
 - 加载、保存、渲染虚拟位图，虚拟位图就是超过GPU的纹理会自动分为多个

- 矢量图

 - 画图形，线、矩形、圆，或使用基础图形组成复杂的
 
 - 使用笔刷、颜色、图形填充图形
 
 - 任意宽度线段


- 图形显影效应


## 使用

打开 vs，创建项目，这里把项目叫 UmmyShirouValeri

打开Nuget，搜索 win2d 安装

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F20173262046.jpg)


打开 MainPage.xaml ，添加命名


```xml
    <Page
    x:Class="UmmyShirouValeri.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:UmmyShirouValeri"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"
    mc:Ignorable="d">
```

添加命名之后，如果需要显示win2d，那么需要使用控件显示


```csharp
            <canvas:CanvasControl x:Name="canvas" ClearColor="Black"></canvas:CanvasControl>

```

先运行一下

一般可以按 F5 运行，如果觉得太早了，出现没有写好的，那么请按 ctrl+break 取消生成

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F20173262081.jpg)

按下F5这时看起来什么没有，但是有了颜色，如果可以看到这个，那么程序是安装成功，如果错误，那么可能安装的包错误

### 添加文字

需要在 canvas 的 Draw 添加函数，可以在这里画出图案，文字，于是` <canvas:CanvasControl x:Name="canvas" ClearColor="Black" Draw="Canvas_OnDraw"></canvas:CanvasControl>` 在 MainPage.xaml.cs 写函数`Canvas_OnDraw`


```csharp
        private void Canvas_OnDraw(Microsoft.Graphics.Canvas.UI.Xaml.CanvasControl sender, Microsoft.Graphics.Canvas.UI.Xaml.CanvasDrawEventArgs args)
        {
            var draw = args.DrawingSession;
            draw.DrawText("lindexi",new Vector2(100,100),Color.FromArgb(0xFF,100,100,100));
        }
```

args.DrawingSession 提供很多方法，可以在这些方法写文字。

如果需要写文字，可以使用 `draw.DrawText` ，方法提供很多参数，一般可以使用这个方法设置显示位置，显示颜色。

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F2017326201422.jpg)

和上面同样功能，可以不使用Vector2，`draw.DrawText("lindexi",100,100,Color.FromArgb(0xFF,100,100,100));`

如果需要设置字体宽度，可以使用


```csharp
            draw.DrawText("lindexi", 100, 100, 500, 50, Color.FromArgb(0xFF, 100, 100, 100), new CanvasTextFormat()
            {
                FontSize = 100
            });
```
![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F2017326201856.jpg)

### 清理

因为 win2d 不会自己清理，一般在页面切换，清理

打开 MainPage.xaml ，添加


```xml
    <Page
    x:Class="UmmyShirouValeri.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:UmmyShirouValeri"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"
    Unloaded="Page_OnUnloaded"
    mc:Ignorable="d">
```

打开  MainPage.xaml.cs ，在函数 Page_OnUnloaded


```csharp
    
        private void Page_OnUnloaded(object sender, RoutedEventArgs e)
        {
            canvas.RemoveFromVisualTree();
            canvas = null;
        }
```

需要记得，这个很重要

为何需要这样，参见：[避免内存泄漏](http://validvoid.net/win2d-avoiding-memory-leaks/)

### 这个标题不知道写什么

搞事，一定要

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F2017%25E5%25B9%25B43%25E6%259C%258827%25E6%2597%25A5%252008005.gif)

那么这是如何做的？

如果需要重新画，如何做？

想要让他重画，使用`canvas.Invalidate();` 就会重新调用Canvas_OnDraw

在构造使用 


```csharp
                DispatcherTimer t=new DispatcherTimer();
            t.Interval=new TimeSpan(1000);
            t.Tick += (s, e) =>
            {
                canvas.Invalidate();
            };
            t.Start();
```

这样就可以隔 1000 毫秒重画


```csharp
            private void Canvas_OnDraw(Microsoft.Graphics.Canvas.UI.Xaml.CanvasControl sender, Microsoft.Graphics.Canvas.UI.Xaml.CanvasDrawEventArgs args)
        {
            var draw = args.DrawingSession;
            draw.DrawText("lindexi", Ran.Next(10,100), Ran.Next(10, 100), 500, 50, Color.FromArgb(0xFF, 100, 100, 100), new CanvasTextFormat()
            {
                FontSize = 100
            });

            draw.DrawLine(Ran.Next(10,100),Ran.Next(10,100),Ran.Next(100,1000),Ran.Next(100,1000),Color.FromArgb(0xFF,25,100,100));
        }

        private Random Ran { set; get; } = new Random();

```

就可以看到上面的效果

是不是觉得还不好看，试试


```csharp
         private void Canvas_OnDraw(Microsoft.Graphics.Canvas.UI.Xaml.CanvasControl sender, Microsoft.Graphics.Canvas.UI.Xaml.CanvasDrawEventArgs args)
        {
            var draw = args.DrawingSession;
            draw.DrawText("lindexi", Ran.Next(10, 100), Ran.Next(10, 100), 500, 50, r(), new CanvasTextFormat()
            {
                FontSize = 100
            });

            for (int i = 0; i < 10; i++)
            {
                draw.DrawLine(Ran.Next(10, 100), Ran.Next(10, 100), Ran.Next(100, 1000), Ran.Next(100, 1000), r());
            }

            Color r()
            {
                return Color.FromArgb(0xFF, rc(), rc(), rc());
            }

            byte rc()
            {
                return (byte)(Ran.NextDouble() * 255);
            }
        }

```

需要在vs2017 才可以跑，如果希望下载vs2017 ，可以到我网盘下载

链接：http://pan.baidu.com/s/1skXDc3z 密码：70d6

如果度盘链接没法使用，请联系我。

btsync：BTZR4YIPCLUUEL2BKDACVGLC3473MEWDN


### 如何画线

刚才已经代码有画线的，也许已经看见

draw.DrawLine（x1，y1，x2，y2，颜色）

除了可以设置线所在的地方，可以设置线条宽度、样式

### 添加图片

添加图片可以`draw.DrawImage` 画出图片，之前需要有图片，需要的是`CanvasBitmap`，如何获得这个？

可以通过


```csharp
    CanvasBitmap.CreateFromBytes()
    CanvasBitmap.CreateFromColors()
    CanvasBitmap.LoadAsync()
```

这些方法得到。

注意：传入的`ICanvasResourceCreator`就是 CanvasControl

下面使用 LoadAsync 传入工程的图片


```csharp
            private void Canvas_OnDrawAsync(Microsoft.Graphics.Canvas.UI.Xaml.CanvasControl sender, Microsoft.Graphics.Canvas.UI.Xaml.CanvasDrawEventArgs args)
        {
            var draw = args.DrawingSession;
            draw.DrawText("lindexi", Ran.Next(10, 100), Ran.Next(10, 100), 500, 50, r(), new CanvasTextFormat()
            {
                FontSize = 100
            });

            for (int i = 0; i < 10; i++)
            {
                draw.DrawLine(Ran.Next(10, 100), Ran.Next(10, 100), Ran.Next(100, 1000), Ran.Next(100, 1000), r());
            }
            if (img != null)
            {
                draw.DrawImage(img, Ran.Next(10, 1000), rc());
            }
            else
            {
                Img().Wait();
            }

            Color r()
            {
                return Color.FromArgb(0xFF, rc(), rc(), rc());
            }

            byte rc()
            {
                return (byte)(Ran.NextDouble() * 255);
            }

            async Task Img()
            {
                img = await CanvasBitmap.LoadAsync(canvas, new Uri("ms-appx:///Assets/SplashScreen.png"));
            }
        }

```


创建图片使用`img = await CanvasBitmap.LoadAsync(canvas, new Uri("ms-appx:///Assets/SplashScreen.png"));`

创建使用的uri参见：[win10 uwp 访问解决方案文件](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E8%AE%BF%E9%97%AE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%96%87%E4%BB%B6/)

## 设置 win2d 背景

win2d 会忽略在 xaml 设置的背景

```csharp
        <xaml:CanvasControl x:Name="canvas" Background="Brown" Draw="Canvas_OnDraw"></xaml:CanvasControl>

```

上面的代码不会把win2d 的背景设置，因为 win2d 需要设置 `ClearColor `

如果想把 win2d 的背景颜色设置为 白色，那么可以使用下面代码

```csharp
        <xaml:CanvasControl x:Name="canvas" ClearColor="White" Draw="Canvas_OnDraw"></xaml:CanvasControl>

```

如果在一次刷新需要设置 win2d 的颜色，那么可以使用 下面代码

```csharp
            sender.ClearColor = Colors.White;

```

## 其他博客

win2d 毛玻璃：[win10 uwp 毛玻璃](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E6%AF%9B%E7%8E%BB%E7%92%83/)

[win2d 画出好看的图形](http://lindexi.oschina.io/lindexi/post/win2d-%E7%94%BB%E5%87%BA%E5%A5%BD%E7%9C%8B%E7%9A%84%E5%9B%BE%E5%BD%A2/)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。