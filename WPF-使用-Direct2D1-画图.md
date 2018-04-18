
# WPF 使用 Direct2D1 画图

本文来告诉大家如何在 WPF 使用 D2D 画图。

<!--more-->


<!-- csdn -->
<!-- 标签：WPF，D2D,DirectX -->

## 什么是 D2D

实际上现在很多小伙伴对于渲染性能就是听到 DirectX 才会去搜索这个博客。我在博客园看到很少的博客讲到这个。即使有也很少会说如何使用 WPF 的。

那么 D2D 是一个提高性能的方法，具体是怎么做？基于 Direct3D 可以使用硬件加速的功能，即使在没有显卡，进行软渲染的性能也是比 GDI 快，但是渲染静态的还是建议使用 GDI。

现在的 WPF 底层使用的渲染是 Dx9 或 Dx11 Dx12 优化  fl9 所以性能实际上和直接使用 D2D 是差不多，但是 WPF 没有充分使用DX，所以如果自己写的性能会比较高。

## Direct2D运行需求

这是我从大神的博客看到，如果需要运行 Direct2D 那么就需要在 win7 之后才可以。

## 安装

下面是我从 Nuget 安装，这个是很老的库，不太建议大家使用。

Nuget 搜索 WindowsAPICodePackDirectX 就可以安装，如果不知道安装哪个，请点击[WindowsAPICodePackDirectX](https://www.nuget.org/packages/WindowsAPICodePackDirectX )

## 环境

如果直接使用这个库是无法运行，下面的代码只是作为大家快速入门，不能用于产品。安装这个库可以用在 x64 但是不能用在 x86 而且需要创建 App.config 文件添加下面代码

```csharp
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <startup useLegacyV2RuntimeActivationPolicy="true">
    <supportedRuntime version="v4.0" sku = ".NETFramework,Version=v4.5"/>
    <supportedRuntime version="v2.0.50727"/>
  </startup>
</configuration>
```


那么如何让软件使用 x64 ，实际上右击项目属性，在生成就可以看到 anycpu 把他修改为 x64 就好了。

## 创建工厂

首先打开 MainPage 的代码，添加下面代码

```csharp
using D2D = Microsoft.WindowsAPICodePack.DirectX.Direct2D1;

```

这样下面就不需要写那么多代码

如果需要使用 Direct2D1 那么就需要先创建工厂。

虽然工厂有很多重载，不过这里不会告诉大家，因为只是快速入门，如果需要知道参数的意思就请自己多看文章。

```csharp
       public MainWindow()
        {
            InitializeComponent();

            Loaded += (s, e) =>
            {
                var d2DFactory = D2D.D2DFactory.CreateFactory(D2D.D2DFactoryType.Multithreaded);
            };
        }
```

## 获得窗口

从上面代码大家也许会说为什么需要在 Load 才写，因为需要拿到窗口，在 Load 之后拿才不会是空

使用下面代码就可以拿到窗口

```csharp
                var windowHandle = new WindowInteropHelper(this).Handle;

```

## 渲染目标

最主要就是创建 RenderTarget ，请使用下面代码

```csharp
var renderTarget = d2DFactory.CreateHwndRenderTarget(new D2D.RenderTargetProperties(),
                    new D2D.HwndRenderTargetProperties(windowHandle,
                        new D2D.SizeU((uint) ActualWidth, (uint) ActualHeight),
                        D2D.PresentOptions.RetainContents));
```

实际上上面的代码不可以用在实际项目，因为写入的大小，如果窗口修改了大小，那么显示的就一般不是期望。

在渲染的时候还需要使用 RenderTarget ，先创建一个字段保存

```csharp
    public MainWindow()
        {
            InitializeComponent();

            Loaded += (s, e) =>
            {
                var d2DFactory = D2D.D2DFactory.CreateFactory(D2D.D2DFactoryType.Multithreaded);

                var windowHandle = new WindowInteropHelper(this).Handle;
                var renderTarget = d2DFactory.CreateHwndRenderTarget(new D2D.RenderTargetProperties(),
                    new D2D.HwndRenderTargetProperties(windowHandle,
                        new D2D.SizeU((uint) ActualWidth, (uint) ActualHeight),
                        D2D.PresentOptions.RetainContents));

                _renderTarget = renderTarget;
            };
        }

        private D2D.RenderTarget _renderTarget;
```

## 写线段

那么尝试对RenderTarget写入线段

因为需要知道在什么时候才进行渲染，所以先添加下面代码

```csharp
            CompositionTarget.Rendering += OnRendering;

```

创建笔刷的方法

```csharp
            var brush = _renderTarget.CreateSolidColorBrush(new D2D.ColorF(0, 1, 0));

```

进行渲染

```csharp
            if (_renderTarget == null)
            {
                return;
            }

            _renderTarget.BeginDraw();
            var brush = _renderTarget.CreateSolidColorBrush(new D2D.ColorF(0, 1, 0));

            _renderTarget.DrawLine(new D2D.Point2F(10, 10), new D2D.Point2F(100, 100), brush, 10);
            
            _renderTarget.EndDraw();
```

尝试运行就可以看到下面界面

![](http://7xqpl8.com1.z0.glb.clouddn.com/lindexi%2F2018418215519377.jpg)

这时看一下 CPU ，几乎不需要。

下面来做很小修改，写出一个会动的图

![](https://i.loli.net/2018/04/18/5ad745c728813.gif)

[Direct2D教程I——简介及首个例子 - 万仓一黍 - 博客园](http://www.cnblogs.com/grenet/p/3249664.html )

所有代码：[WPF Direct2D 入门-CSDN下载](https://download.csdn.net/download/lindexi_gd/10358588 )

```csharp
       public MainWindow()
        {
            InitializeComponent();

            CompositionTarget.Rendering += OnRendering;

            Loaded += (s, e) =>
            {
                var d2DFactory = D2D.D2DFactory.CreateFactory(D2D.D2DFactoryType.Multithreaded);

                var windowHandle = new WindowInteropHelper(this).Handle;
                var renderTarget = d2DFactory.CreateHwndRenderTarget(new D2D.RenderTargetProperties(),
                    new D2D.HwndRenderTargetProperties(windowHandle,
                        new D2D.SizeU((uint) ActualWidth, (uint) ActualHeight),
                        D2D.PresentOptions.RetainContents));

                _redBrush = renderTarget.CreateSolidColorBrush(new D2D.ColorF(1, 0, 0, 1));

                _greenBrush = renderTarget.CreateSolidColorBrush(new D2D.ColorF(0, 1, 0, 1));

                _blueBrush = renderTarget.CreateSolidColorBrush(new D2D.ColorF(0, 0, 1, 1));

                _renderTarget = renderTarget;
            };
        }

        private D2D.RenderTarget _renderTarget;

        private D2D.SolidColorBrush _redBrush;

        private D2D.SolidColorBrush _greenBrush;

        private D2D.SolidColorBrush _blueBrush;

        private void OnRendering(object sender, EventArgs e)
        {
            if (_renderTarget == null)
            {
                return;
            }

            D2D.SolidColorBrush brush = null;

            switch (ran.Next(3))
            {
                case 0:
                    brush = _redBrush;
                    break;
                case 1:
                    brush = _greenBrush;
                    break;
                case 2:
                    brush = _blueBrush;
                    break;
            }

            _renderTarget.BeginDraw();
            _renderTarget.DrawRectangle(new D2D.RectF(_x, _y, _x + 10, _y + 10), brush, 1);
            _renderTarget.EndDraw();

            _x = _x + _dx;
            _y = _y + _dy;
            if (_x >= ActualWidth - 100 || _x <= 0)
            {
                _dx = -_dx;
            }

            if (_y >= ActualHeight - 100 || _y <= 0)
            {
                _dy = -_dy;
            }
        }

        private float _dx = 1;
        private float _dy = 1;
        private float _x;
        private float _y;

        private Random ran = new Random();
```

## 更多博客

[为何使用 DirectComposition](https://lindexi.oschina.io/lindexi/post/%E4%B8%BA%E4%BD%95%E4%BD%BF%E7%94%A8-DirectComposition.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。