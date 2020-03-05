# win10 uwp 渲染原理 DirectComposition 渲染

本文来告诉大家一个新的技术[DirectComposition](https://msdn.microsoft.com/zh-cn/library/windows/desktop/hh437376.aspx )，在 win7 之后（实际上是 vista），微软正在考虑一个新的渲染机制。

<!--more-->
<!-- CreateTime:2019/3/8 9:18:16 -->

<div id="toc"></div>

<!-- 标签：uwp,DirectComposition，win2d,渲染 -->

在 Windows  Vista 就引入了一个服务，桌面窗口管理器[Desktop Window Manager](https://msdn.microsoft.com/en-us/library/windows/desktop/aa969540(v=vs.85).aspx )，虽然从[借助 C++ 进行 Windows 开发](https://msdn.microsoft.com/magazine/dn745861 )博客可以看到 DWM 不是一个好的方法，但是比之前好。

在 win8 的时候，微软提出了 DirectComposition ，这是一个新的方法。

在软件的渲染一直都是两个阵营，一个是使用直接渲染模式。直接渲染的例子是使用 Direct2D 和 Direct3D ，而直接通过 Dx api 的方式当然需要使用 C++ 和底层的 API ，这开发效率比较差。

在 1511 发布，微软告诉大家可以使用底层的 DirectComposition 接口，这样大家就可以通过 DirectComposition 做出好看的效果

在原来的 UWP 应用，大家很容易使用 xaml 来写一个界面，但是如果没有 xaml 那么如何创建一个界面。

我不会告诉大家去 new 一个控件，因为这样和使用之前的方法差不多。我会告诉大家如何从一个 Visual 开始画。

在 UWP 可以通过下面几个方式显示界面

 - 通过 xaml 或者后台新建控件显示。这是最推荐的方法，本文下面的方法是不推荐的，但是可以让大家知道原理。使用 xaml 显示的元素一般都是继承 UIElement ，创建出来的元素可以带交互。

 - 如果需要高性能的画图，通过 win2d 是一个很好的方法。大家也知道创建的win2d只是显示，不会有交互，如果需要交互需要自己写。虽然写一个交互很简单，但是如果没有使用框架，重复代码很多。

 - 使用 DirectX APIs 来画 3d 的图片，但是现在需要一些 C++ 代码。

在 UWP 的显示，推荐使用 xaml 来写界面，原因是 xaml 是一个界面无关的代码，也就是无论是 C# 和 C++ 都可以使用。如果使用 C# 来写界面，那么代码就和 C# 合在一起，不能很好在 C++ 运行。而且使用xaml 写简单比使用C#更简单，在 vs 实时编译器可以看到界面效果。

也许大家会关系 fds 是如何做出来的，对于微软的设计，所有的 xaml 或者 win2d 的显示都是位图。这里的位图不是大家想的 bitmapImage 而是显示的一个说法，微软对所有的位图输出到 DirectComposition 。微软的 DirectComposition 在官方是这样说 “DirectComposition 组件使开发者能够进行高性能的位图合成，并附加变换、特效以及动画等各种效果，以此打造出更为复杂、生动、流畅的用户界面。DirectComposition 利用图形硬件的加速特性可以进行与 UI 线程无关的渲染处理，支持 2D 仿射变换、3D 透视变换等多种变换，以及剪切、不透明等基本特效”。

翻译参见 [Windows Composition API 指南 - 认识 Composition API](https://validvoid.net/windows-composition-api-guide-introduction/ ) 感谢大神。

那么是不是可以通过Composition显示元素，自己来写 UWP 框架。

在开始告诉大家写 UWP 框架之前，先给大家一个简单的例子，如何应用 DirectComposition 的 API 写出界面框架。需要知道 DirectComposition 虽然很好用，但是开发的技术要求是 C++ 和 COM 开发难度很高，在 fall creators update 16299 以上的版本，可以使用 Windows.UI.Composition 的方法，可以直接在 xaml 中写出调用 DirectComposition 的方法，同时后台代码可以使用 C# 写

虽然本文想直接告诉小伙伴如何使用 C++ 和 COM 写一个 DirectComposition 的应用，但是因为发现难度太大了，同时微软也建议小伙伴使用 Windows.UI.Composition 而不是使用 DirectComposition 写应用，所以下面将告诉大家如何使用 Windows.UI.Composition 从零开始写一个应用

## 例子

之前写的一个简单的动画是一个好看效果，请看 [win10 uwp 进度条 WaveProgressControl](https://lindexi.gitee.io/post/win10-uwp-%E8%BF%9B%E5%BA%A6%E6%9D%A1-WaveProgressControl.html )

下面来通过删除所有 xaml 文件，从头自己写。

## 创建工程

首先创建一个 UWP 项目，注意选择比较高的目标。最低支持要求是 16299 的系统，这里的 16299 指的是系统版本 

![](http://image.acmx.xyz/lindexi%2F20184221150375871.jpg)

## 如何写显示

现在创建项目，删除所有的 app 和 mainpage 类。重新创建一个类。

只要支持显示，那么就可以完成一半了，因为 UWP 的元素显示都是通过布局找到元素显示的位置。当然这里不会提到 Translate 等。然后元素通过调用DrawContex告诉显卡需要显然的图形。然后在加上用户的输入，就构成了框架。

虽然一个框架比上面说的复杂很多，但是在写 Avalonial 的时候，大神告诉我，实际上一个界面框架主要的就是显示和交互。本文不会告诉大家如何写交互，只是告诉大家如何显示。

删除了所有的自动生成的代码，现在创建一个类 View ，用来显示。

下面代码的意思请看 [【Win 10 应用开发】UI Composition 札记（一）：视图框架的实现 - 东邪独孤 - 博客园](http://www.cnblogs.com/tcjiaan/p/7765444.html )

```csharp
using System.Numerics;
using Windows.ApplicationModel.Core;
using Windows.UI;
using Windows.UI.Composition;
using Windows.UI.Core;

namespace HmeucHsvv
{
    internal class View : IFrameworkView, IFrameworkViewSource
    {
        public void SetWindow(CoreWindow window)
        {
            _compositor = new Compositor();

            _compositionTarget = _compositor.CreateTargetForCurrentView();

            // 创建一个容器，用来向他的 Children 添加 Visual 显示复杂的元素
            var container = _compositor.CreateContainerVisual();
            _compositionTarget.Root = container;

            // 创建 SpriteVisual ，这个类不仅是一个容器，同时本身也是可以画出来
            var visual = _compositor.CreateSpriteVisual();

            // 告诉这个元素的大小和左上角，所以这个就是一个矩形，而且设置颜色
            visual.Size = new Vector2(100, 100);
            visual.Offset = new Vector3(10, 10, 0);

            visual.Brush = _compositor.CreateColorBrush(Colors.Red);

            // 添加元素，添加进去的元素就会被显示
            container.Children.InsertAtTop(visual);
        }

        public void Run()
        {
            //启动窗口需要激活
            var window = CoreWindow.GetForCurrentThread();
            window.Activate();
            //调度方式使用 Dispatcher 通过这个就可以获得消息
            window.Dispatcher.ProcessEvents(CoreProcessEventsOption.ProcessUntilQuit);
        }

        public void Initialize(CoreApplicationView applicationView)
        {
        }

        public void Load(string entryPoint)
        {
        }

        public void Uninitialize()
        {
        }

        public IFrameworkView CreateView()
        {
            return this;
        }

        private CompositionTarget _compositionTarget;
        private Compositor _compositor;

        private static void Main()
        {
            CoreApplication.Run(new View());
        }
    }
}
```

上面代码有一些注释，通过这个方式就可以创建一个显示矩形

![](http://image.acmx.xyz/lindexi%2F20184222018324337.jpg)

实际上从上面代码很容易就知道，只需要一个类继承`IFrameworkView, IFrameworkViewSource`，然后使用`CreateView`返回他自己，这时这个类就可以显示。

但是还需要使用主函数告诉软件启动的类是哪个，在运行启动窗口，如果注释掉`window.Activate`那么就会看到只有一个欢迎的图片不会显示矩形。

那么是什么时候窗口支持渲染的？

核心代码是`CreateTargetForCurrentView`这个函数只能调用一次，如果你尝试调用他两次，那么就会出现异常。因为调用这个函数就会告诉 DirectComposition 创建元素。

```csharp
           _compositor = new Compositor();

            _compositionTarget = _compositor.CreateTargetForCurrentView();
```

显示的矩形是通过创建 SpriteVisual 来显示。那么下面再写一个 SpriteVisual ，让两个加起来。

```csharp
        public void SetWindow(CoreWindow window)
        {
            _compositor = new Compositor();

            _compositionTarget = _compositor.CreateTargetForCurrentView();

            // 创建一个容器，用来向他的 Children 添加 Visual 显示复杂的元素
            var container = _compositor.CreateContainerVisual();
            _compositionTarget.Root = container;

            // 创建 SpriteVisual ，这个类不仅是一个容器，同时本身也是可以画出来
            var visual = _compositor.CreateSpriteVisual();

            // 告诉这个元素的大小和左上角，所以这个就是一个矩形，而且设置颜色
            visual.Size = new Vector2(100, 100);
            visual.Offset = new Vector3(10, 10, 0);

            visual.Brush = _compositor.CreateColorBrush(Colors.Red);

            // 添加元素，添加进去的元素就会被显示
            container.Children.InsertAtTop(visual);

            
            var visual1 = _compositor.CreateSpriteVisual();

            // 创建一个重叠元素
            visual1.Size = new Vector2(100, 100);
            visual1.Offset = new Vector3(20, 20, 0);

            visual1.Brush = _compositor.CreateColorBrush(
                Color.FromArgb(128 /*透明*/, 0, 255, 0));
            container.Children.InsertAtTop(visual1);
        }

```

使用这个方法就可以创建多个矩形，而且通过指定位置就和大小就可以决定他在哪显示。

上面用到了三个东西第一个是 Visual ，这是一个基础的类。有 ContainerVisual 继承 Visual ，实际上他只是可以存在子元素。最后一个是 SpriteVisual ，这个类和 ContainerVisual 一样，但是他可以使用笔刷。

那么 SpriteVisual 设置的笔刷是什么，他可以设置三个不同的笔刷。第一个就是刚才给大家看的 CompositionColorBrush ，这是一个纯色笔刷。 第二个是比较复杂的，可以使用特效的 CompositionEffectBrush 笔刷，最后一个是 CompositionSurfaceBrush 可以和 dx 交互数据。

![](http://image.acmx.xyz/lindexi%2F20184222024298849.jpg)

从上面代码实际只是画了普通的矩形，如果要写文字，画线，那么怎么办。这时就需要使用 CompositionSurfaceBrush ，这是最复杂的。通过这个类可以使用 d2d 来画，在 UWP 简单使用的方法是 win2d 所以下面告诉大家如何使用 win2d 来画。

但是 UWP 底层是直接使用d2d没有经过 win2d 的封装。从我的博客[WPF 使用 SharpDX 在 D3DImage 显示](https://lindexi.gitee.io/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX-%E5%9C%A8-D3DImage-%E6%98%BE%E7%A4%BA.html )可以知道，在 WPF 使用 d2d 是比较难的，因为很难集合两个在一个界面。但是 UWP 通过这个类就可以把底层渲染放在指定层级。这就是为什么说 UWP 可以做出比较高性能，因为 WPF 是很难修改他的渲染，即使使用D3DImage也是把渲染位图作为图片显示，需要先在显卡渲染然后把位图复制到内存，让WPF画出图片。但是 UWP 可以直接画出，不需要使用 WPF 这样的方法。我看来 UWP 在这里是很大提升，这就是我看到很多大神说不在 WPF 添加 win2d ，从底层技术实现是不相同。

## CompositionSurfaceBrush

首先需要安装 win2d ，然后在 SetWindow 使用 CompositionSurfaceBrush 。还是和上面代码一样，但是需要先创建一个函数，用来创建 win2d ，请看下面

```csharp
        private void GetCanvasAndGraphicsDevices()
        {
            var canvasDevice = CanvasDevice.GetSharedDevice();

            _graphicsDevice = CanvasComposition.CreateCompositionGraphicsDevice(
                _compositor, canvasDevice);

            _graphicsDevice.RenderingDeviceReplaced += OnRenderingDeviceReplaced;
        }
```

通过这个方法就可以拿到 graphicsDevice ，这个就是用来做 CompositionSurfaceBrush 。

如果需要创建 CompositionSurfaceBrush 那么就需要一个 CompositionDrawingSurface ，而 CompositionDrawingSurface 可以通过 graphicsDevice 创建，代码很简单

```csharp
            _compositor = new Compositor();

            _compositionTarget = _compositor.CreateTargetForCurrentView();

            // 创建 win2d 用于渲染
            GetCanvasAndGraphicsDevices();

            _drawingSurface = _graphicsDevice.CreateDrawingSurface(
                new Size(600, 600),
                DirectXPixelFormat.B8G8R8A8UIntNormalized,
                DirectXAlphaMode.Premultiplied);

            var brush = _compositor.CreateSurfaceBrush(
                _drawingSurface);
```

那么创建的 CompositionSurfaceBrush 如何显示？刚才讲到SpriteVisual可以显示笔刷，那么就创建这个类来显示。

```csharp

            var drawingVisual = _compositor.CreateSpriteVisual();
            drawingVisual.Size = new Vector2(600, 600);

            drawingVisual.Brush = brush;
```

然后把他加入视觉，和上面的代码一样，只是把 Brush 的创建写了其他的代码

```csharp
            var containerVisual = _compositor.CreateContainerVisual();
            _compositionTarget.Root = containerVisual;

            containerVisual.Children.InsertAtTop(drawingVisual);
```

下面就是让 win2d 画出矩形。

```csharp
        private void Redraw()
        {
            using (var drawingSession = CanvasComposition.CreateDrawingSession(
                _drawingSurface))
            {
                drawingSession.FillRectangle(
                    new Rect(10, 10, 200, 200),
                    Colors.Red);

                drawingSession.FillRectangle(
                    new Rect(300, 300, 200, 200),
                    Color.FromArgb(255,126,50,50));
            }
        }
```

什么时候可以调用这个函数？实际上在刚才的函数最后调用就可以了。

现在的界面就是两个矩形

![](http://image.acmx.xyz/lindexi%2F20184222047529494.jpg)

所有的代码

```csharp
    internal class View : IFrameworkView, IFrameworkViewSource
    {
        public void SetWindow(CoreWindow window)
        {
            _compositor = new Compositor();

            _compositionTarget = _compositor.CreateTargetForCurrentView();

            // 创建 win2d 用于渲染
            GetCanvasAndGraphicsDevices();

            _drawingSurface = _graphicsDevice.CreateDrawingSurface(
                new Size(600, 600),
                DirectXPixelFormat.B8G8R8A8UIntNormalized,
                DirectXAlphaMode.Premultiplied);

            var brush = _compositor.CreateSurfaceBrush(
                _drawingSurface);

            var drawingVisual = _compositor.CreateSpriteVisual();
            drawingVisual.Size = new Vector2(600, 600);

            drawingVisual.Brush = brush;


            var containerVisual = _compositor.CreateContainerVisual();
            _compositionTarget.Root = containerVisual;

            containerVisual.Children.InsertAtTop(drawingVisual);

            Redraw();
        }

        public void Run()
        {
            //启动窗口需要激活
            var window = CoreWindow.GetForCurrentThread();
            window.Activate();
            //调度方式使用 Dispatcher 通过这个就可以获得消息
            window.Dispatcher.ProcessEvents(CoreProcessEventsOption.ProcessUntilQuit);
        }

        public void Initialize(CoreApplicationView applicationView)
        {
        }

        public void Load(string entryPoint)
        {
        }

        public void Uninitialize()
        {
        }

        public IFrameworkView CreateView()
        {
            return this;
        }

        private CompositionTarget _compositionTarget;
        private Compositor _compositor;
        private CompositionGraphicsDevice _graphicsDevice;
        private CompositionDrawingSurface _drawingSurface;

        private static void Main()
        {
            CoreApplication.Run(new View());
        }

        private void GetCanvasAndGraphicsDevices()
        {
            var canvasDevice = CanvasDevice.GetSharedDevice();

            _graphicsDevice = CanvasComposition.CreateCompositionGraphicsDevice(
                _compositor, canvasDevice);

            //_graphicsDevice.RenderingDeviceReplaced += OnRenderingDeviceReplaced;
        }

        private void OnRenderingDeviceReplaced(
            CompositionGraphicsDevice sender, RenderingDeviceReplacedEventArgs args)
        {
            Redraw();
        }

        private void Redraw()
        {
            using (var drawingSession = CanvasComposition.CreateDrawingSession(
                _drawingSurface))
            {
                drawingSession.FillRectangle(
                    new Rect(10, 10, 200, 200),
                    Colors.Red);

                drawingSession.FillRectangle(
                    new Rect(300, 300, 200, 200),
                    Color.FromArgb(255,126,50,50));
            }
        }
    }

```

那么尝试使用 win2d 写文字就请看[win10 uwp win2d 入门 看这一篇就够了](https://lindexi.gitee.io/post/win10-uwp-win2d-%E5%85%A5%E9%97%A8-%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86.html )

修改函数和普通使用 win2d 没有不同

```csharp
        using (var drawingSession = CanvasComposition.CreateDrawingSession(
                _drawingSurface))
            {
                drawingSession.Clear(Colors.White);
                drawingSession.DrawText("lindexi", new Vector2(100, 100), Color.FromArgb(0xFF, 100, 100, 100));
            }
```

![](http://image.acmx.xyz/lindexi%2F20184222056348323.jpg)

还有如何使用动画和特效，我这里就不说了。

代码参考 [图形和动画 - Windows 组合支持 10 倍缩放](https://msdn.microsoft.com/zh-CN/magazine/mt590968 )

参考：

[图形和动画 - Windows 组合支持 10 倍缩放](https://msdn.microsoft.com/zh-CN/magazine/mt590968 )

[【Win 10 应用开发】UI Composition 札记（一）：视图框架的实现 - 东邪独孤 - 博客园](http://www.cnblogs.com/tcjiaan/p/7765444.html )

[借助 C++ 进行 Windows 开发 - 使用 Windows 组合引擎实现高性能窗口分层](https://msdn.microsoft.com/magazine/dn745861 )

[借助 C++ 进行 Windows 开发 - 使用 Windows 组合引擎](https://msdn.microsoft.com/magazine/dn786854 )

[Windows, UI and Composition (the Visual Layer) – Mike Taulty](https://mtaulty.com/2015/12/17/m_15996/ )

[Windows with C++ - DirectComposition: A Retained-Mode API to Rule Them All](https://msdn.microsoft.com/en-us/magazine/dn759437.aspx )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
