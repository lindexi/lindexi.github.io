
# MAUI 框架开发 将 MAUI 嵌入到 WPF 控件里

本文将介绍如何将 MAUI 的底层替换为 WPF 框架层，且将 MAUI 的内容嵌入到 WPF 的一个控件里面，无 UI 框架嵌入的空域问题

<!--more-->


<!-- csdn -->
<!-- 博客 -->
<!-- 发布 -->

本文是 MAUI 框架开发博客，而不是 MAUI 应用开发博客，本文更多介绍的是进行 MAUI 这个框架的开发内容。不熟悉或不进行 MAUI 框架开发的伙伴也可以看着玩，看看这个有趣的黑科技。必须说明的是本文介绍的这条路仅仅只是我的想法，本文也仅仅完成了证明了技术上的可行性，不代表着后续 MAUI 必须往这个方向发展，以及不代表工程上的可行性

开始之前先看看效果，以下代码是放入到 WPF 项目的 MainWindow.xaml 文件里面的

```xml
    <Grid>
        <StackPanel>
            <Canvas x:Name="MauiMainPageCanvas" Width="1000" Height="500"></Canvas>
            <Button HorizontalAlignment="Center">Wpf Button</Button>
        </StackPanel>
        <Border Width="600" Height="50" Background="Blue" HorizontalAlignment="Left">
            <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center">Not Airspace issues</TextBlock>
            <Border.RenderTransform>
                <RotateTransform Angle="90" CenterX="300" CenterY="0"></RotateTransform>
            </Border.RenderTransform>
        </Border>
    </Grid>
```

以上代码的 MauiMainPageCanvas 就是一个用来承载 Maui 的 MainPage 的 Canvas 控件。接下来的 Maui 的 MainPage 界面将会在此显示。以上代码表现了此方案可以支持将 MAUI 的内容嵌入到 WPF 的一个 Canvas 控件里面，且受到 WPF 布局的约束，如放入到 StackPanel 里面被布局。接着下方放一个带旋转的 Border 控件，一半覆盖住了 MauiMainPageCanvas 控件，用来表示没有空域问题。假定有空域问题，那大家跑起来一眼就能看出来了

以下的代码是放入到 MAUI 项目里面，代码是放入到 MAUI 项目的 MainPage.xaml 里面，是一个简单的按钮加上背景设置一点颜色

```csharp
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MauiApp.MainPage">

    <VerticalStackLayout
        Spacing="25"
        Padding="30,0"
        VerticalOptions="Center"
        HeightRequest="500"
        BackgroundColor="#A0565656">

        <Button
            x:Name="CounterButton"
            Text="Click me"
            Clicked="OnCounterClicked"
            HorizontalOptions="Center" />

    </VerticalStackLayout>

</ContentPage>
```

如此预期的显示就是 WPF 里面的 Canvas 显示出 MAUI 的 MainPage 的界面内容，同时以上的 MAUI 的 CounterButton 还加上了 OnCounterClicked 点击事件，在点击事件里面修改了按钮的文本内容，如以下代码

```csharp
    private int _count = 0;

    private void OnCounterClicked(object sender, EventArgs e)
    {
        _count++;

        if (_count == 1)
            CounterButton.Text = $"Clicked {_count} time";
        else
            CounterButton.Text = $"Clicked {_count} times";
    }
```

预期就是交互上点击 MAUI 的按钮，可以看到按钮的文本变更了，这就证明了 MAUI 整个上层逻辑是可以符合预期工作的

跑起来怎样呢？我提供代码，大家可以自己跑跑看

以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/efdae121a59c438323155a5de825937e9c686cd2/MauiForWpf_CikerenearkohereWhefaljearnu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/efdae121a59c438323155a5de825937e9c686cd2/MauiForWpf_CikerenearkohereWhefaljearnu) 欢迎访问

可以通过如下方式获取本文以上的源代码，先创建一个名为 MauiForWpf_CikerenearkohereWhefaljearnu 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin efdae121a59c438323155a5de825937e9c686cd2
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin efdae121a59c438323155a5de825937e9c686cd2
```

获取代码之后，进入 MauiForWpf_CikerenearkohereWhefaljearnu 文件夹

跑起来的方法就是设置 WpfApp 为启动项目，然后一键 F5 即可跑起来。当然，别忘了 VisualStudio 2022 打全负载哦

接下来是原理部分

在 MAUI 里面，从大的设计上，整个 MAUI 处于 UI 框架的上层，且中间提供了 `Microsoft.Maui` 这个 Core 层，这个抽象约定层。理论上任何实现了 Core 层约定的抽象定义的项目，都可以作为 MAUI 的 UI 框架的底层组件。显然在 WPF 里面是完全有能力做到实现好 MAUI 的 Core 层抽象层定义的，也就是 WPF 完全可以作为 MAUI 的底层。在 MAUI 设计之处，本身 WPF 就是设计在 MAUI 的底层里面

那既然本身就有这个设计了，我这篇博客不就是完全抄写官方的设计了？其实不然，按照官方的设计是 MAUI 作为整个完整项目的存在。而本文提供的黑科技是让 MAUI 作为其他的 WPF 项目里面的一个控件的存在。这就有趣的很了，试想，我现在有一个成熟的 WPF 项目。但是我想玩玩 MAUI 应用开发，可以怎么办呢？最佳的办法就是这个项目里面有部分模块，部分界面采用 MAUI 编写。可以让 MAUI 编写界面里面其中某些控件，这样既不需要大改现有项目，也没有什么迁移成本。还可以一边开发 WPF 的同时开发 MAUI 项目

从这个角度上看，本文的这个玩法就似乎超过了 MAUI 的初始设计了？其实没有哈，我的这个想法其实也是从 MAUI 其中一个设计会议上听来的，当时没有记下是哪位大佬的提议，但我感觉特别有可行性。刚好最近放暑假了，有点点空闲余力，而且从 AIGC 项目的预研上让我不小心理解了 MAUI 框架的设计的重要部分，于是我尝试成功了在不更改 MAUI 基础框架的前提下，只编写上层代码，实现将 WPF 框架注入到 MAUI 框架里面，让 WPF 作为 MAUI 框架的底层，且支持 MAUI 项目的某个部分，如 MainPage 嵌入到 WPF 的某个控件上

以下是此黑科技的实现方法，我新建了三个项目，分别是 MauiApp 和 MauiWpfAdapt 和 WpfApp 项目，从命名上大家也可以看出来这三个项目分别的功能。其中 MauiApp 是一个 MAUI 项目，完全的 MAUI 项目的实现，没有掺杂任何的黑科技，十分良心，嗯，不好意思，串场了。而 WpfApp 则是一个非常纯粹的 WPF 项目，除了引用 MauiWpfAdapt 项目外，也没有掺杂任何的黑科技，都是纯净的 WPF 实现

那中间的 MauiWpfAdapt 项目是引用了 MauiApp 项目，然后被 WpfApp 项目引用的。既然前后两个项目都没有掺杂黑科技，那黑科技自然只能落到 MauiWpfAdapt 项目里面了。从命名上可以知道，这个 MauiWpfAdapt 项目是一个适配器形的项目，功能上就是让 MAUI 能够作为 WPF 的一个控件嵌入到 WPF 项目里面

这个 MauiWpfAdapt 项目提供了 MauiForWpfHostHelper 帮助类，代码方法签名如下

```csharp
public static class MauiForWpfHostHelper
{
    public static MauiApplicationProxy InitMauiApplication(System.Windows.Application wpfApplication);
    public static void HostMainPage(Panel panel, MauiApplicationProxy applicationProxy);
}
```

调用方是放在 WPF 项目里面，在 App.xaml.cs 里面调用 MauiForWpfHostHelper 的 InitMauiApplication 方法进行初始化。在 MainWindow.xaml.cs 里调用 MauiForWpfHostHelper 的 HostMainPage 方法将 MauiApp 的 MainPage 进行嵌入

以下是 App.xaml.cs 里面的代码

```csharp
public partial class App : Application
{
    public App()
    {
        var mauiApplicationProxy = MauiForWpfHostHelper.InitMauiApplication(this);
        MauiApplicationProxy = mauiApplicationProxy;
    }

    public MauiApplicationProxy MauiApplicationProxy { get; }
}
```

以下是 MainWindow.xaml.cs 里面的代码

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        MauiForWpfHostHelper.HostMainPage(MauiMainPageCanvas, ((App) Application.Current).MauiApplicationProxy);
    }
}
```

如此可以看出来，在 Wpf 项目里面的使用方法是非常简单的。复杂的细节都放在 MauiWpfAdapt 里面。先看看 MauiForWpfHostHelper 的 InitMauiApplication 方法做了哪些黑科技

```csharp
public static class MauiForWpfHostHelper
{
    public static MauiApplicationProxy InitMauiApplication(System.Windows.Application wpfApplication)
    {
        var builder = MauiProgram.CreateMauiAppBuilder();

        builder.Services.ConfigureMauiHandlers(collection =>
        {
            collection.AddHandler<Application, FooApplicationHandler>();
            collection.AddHandler<Microsoft.Maui.Controls.Window, FooWindowHandler>();
            collection.AddHandler<Page, FooPageHandler>();
            collection.AddHandler<Layout, FooLayoutHandler>();
            collection.AddHandler<Button, FooButtonHandler>();
        });

        // 这是一个标记过时的类型，只是 MAUI 为了兼容之前的坑而已，后续版本将会删除
        DependencyService.Register<ISystemResourcesProvider, ObsoleteSystemResourcesProvider>();

        var mauiApp = builder.Build();

        var rootContext = new FooMauiContext(wpfApplication, mauiApp.Services);

        var app = mauiApp.Services.GetRequiredService<IApplication>();
        app.SetApplicationHandler(app, rootContext);
        var fooApplicationHandler = (FooApplicationHandler) app.Handler;
        _ = fooApplicationHandler;

        return new MauiApplicationProxy(app);
    }

    ... // 忽略其他代码
}
```

可以看到在 InitMauiApplication 方法里面大量进行了注入 FooXxxxHandler 模块。而且还将 Wpf 的 `System.Windows.Application` 构成 FooMauiContext 上下文设置到 Maui 的 App 处理器里面

从代码可以看到，精髓就在各个 FooXxxxHandler 模块里面。在 Maui 的大的框架设计里面，是采用组件化模式设计，配合中央容器进行装配注入。如此可以实现各个模块都可以自定义替换

还有一个小细节是替换模块时需要自定义的自定义模块是可以继承原有的模块的，如此可以省下不少的开发工作量。以下拿 Button 举例子，通过编写 `class FooButtonHandler : ButtonHandler` 即可让 FooButtonHandler 继承默认的 ButtonHandler 实现。由于当前咱采用的是 WPF 框架作为底层框架，现在 2023 还没有可用的默认实现，所继承的 ButtonHandler 里面都是空白的实现，也就是每个实现方法大部分都是啥都不做

继续使用按钮的处理器作为例子，咱可以重写 CreatePlatformView 方法，让其返回 WPF 的按钮控件，如以下代码

```csharp
class FooButtonHandler : ButtonHandler
{
    protected override object CreatePlatformView()
    {
        return new System.Windows.Controls.Button();
    }

    ... // 忽略其他代码
}
```

想要聊以上的 CreatePlatformView 的方法，就必须说到在 MAUI 里面，咱定义的业务上的 MAUI 的控件，在 MAUI 框架里面都称为 VirtualView 虚拟的控件，这是和 PlatformView 平台的控件相对应。在 MAUI 里面，所有的控件都是浮在底层 UI 框架上方的，每个控件都可以由底层 UI 托管为真正的平台实现。换句话说就是 MAUI 的跨平台就是每个平台自己实现一套 MAUI 的底层，平台实现部分的对接控件就称为 PlatformView 平台的控件

接下来继续重写 GetDesiredSize 和 PlatformArrange 用于对接 MAUI 的布局框架以及平台的布局框架。在 MAUI 里面有默认的 CrossPlatformMeasure 和 CrossPlatformArrange 方法，这就意味着具体平台需要编写的只是将 MAUI 布局层布局好的控件放入到平台对应的位置即可。重写的 GetDesiredSize 是用来告诉 MAUI 布局框架层，当前的控件的实际尺寸。而重写的 PlatformArrange 则是根据 MAUI 的布局层算好的范围执行将平台控件放入到平台框架正确的坐标

```csharp
class FooButtonHandler : ButtonHandler
{
    public override Size GetDesiredSize(double widthConstraint, double heightConstraint)
    {
        Button.Measure(new System.Windows.Size(widthConstraint, heightConstraint));
        return new Size(Button.DesiredSize.Width + 100, Button.DesiredSize.Height);
    }

    public override void PlatformArrange(Rect rect)
    {
        base.PlatformArrange(rect);

        Button.SetValue(Canvas.LeftProperty, rect.Left);
        Button.SetValue(Canvas.TopProperty, rect.Top);

        Button.Width = rect.Width;
        Button.Height = rect.Height;
    }

    ... // 忽略其他代码
}
```

需要说明的是以上的 FooButtonHandler 实现仅仅只是用来样式，虽然距离真正可用不远，但依然不推荐大家直接抄到实际项目里面

在 MAUI 里面的 Button 控件是可以通过 Text 属性设置按钮的文本的。咱如果想要在 WPF 平台实现上也让按钮支持 MAUI 的按钮功能就需要输入属性重写逻辑，如以下代码

```csharp
class FooButtonHandler : ButtonHandler
{
    public FooButtonHandler() : base(new PropertyMapper<IButton, IButtonHandler>(ButtonHandler.Mapper)
    {
        [nameof(IText.Text)] = MapFooText
    })
    {
    }

    private static void MapFooText(IButtonHandler buttonHandler, IButton button)
    {
        var fooButtonHandler = (FooButtonHandler) buttonHandler;
        if (button is IText text)
        {
            fooButtonHandler.Button.Content = text.Text;
        }
    }

    private System.Windows.Controls.Button Button => (System.Windows.Controls.Button) PlatformView;

    ... // 忽略其他代码
}
```

在 ButtonHandler 的构造函数里面，允许传入对属性的处理。这里传入的是在原有的 `ButtonHandler.Mapper` 基础上，覆盖或追加对 `IText.Text` 属性变更的处理。在 MapFooText 里面就是对按钮的 Text 属性进行处理的逻辑，这个 MapFooText 方法会在 MAUI 的 Button 按钮初始化完成之后调用，以及后续的任何对 MAUI 的 Button 按钮的 Text 属性变更的时候触发

在 MapFooText 将 MAUI 的 Button 按钮的 Text 属性赋值给到 WPF 的 Button 的内容，如此即可让 WPF 的按钮呈现设置在 MAUI 的 Button 按钮的文本

通过以上的例子也可以看出 MAUI 是可以支持各个平台对相同的 MAUI 的控件的属性有不同的解释，如此属于跨平台框架实现的一个选择，那就是让各个平台保持各个平台的特性。这样的做法的优点在于可以更大程度保留各个平台的功能，同时平台实现本身的性能也不差，相比全自绘来说可以使用到更高的平台性能

作为微软家的设计师，在设计 MAUI 的时候，怎么只会在跨平台框架实现上只采用一个选择呢？微软家的设计师可是都要的哦。在以上的基础上，如果想要让各个平台行为相同，那自然就不能保持其平台特性了。想想，对于小团队来说，没有足够的开发精力去测试各个平台的差异性，此时更多的想法是让各个平台的行为保持一致，虽然 App 写的一般般可也不会挖坑。如果想要让各个平台保持相同的行为，这时就可以采用 MAUI 的统一渲染层来实现。这也是 MAUI 一开始就设计进去的大功能。但是必须说明的是这个设计虽然很好，但也相当相当费开发者，显然现在 MAUI 开发团队还不能完成这个设计的工作

通过注入对 MAUI 的 Button 按钮的 Text 属性的处理即可实现显示 MAUI 按钮上的文本。那如何在用户点击按钮时，回过来触发到 MAUI 按钮的点击逻辑呢？这时就需要平台层主动处理交互逻辑，如以下代码，重写连接函数，监听 WPF 按钮的点击事件，将点击事件给到 MAUI 的按钮的点击

```csharp
class FooButtonHandler : ButtonHandler
{
    protected override void ConnectHandler(object platformView)
    {
        var button = (System.Windows.Controls.Button) platformView;
        button.Click += OnClick;
    }

    private void OnClick(object sender, RoutedEventArgs e)
    {
        VirtualView.Clicked();
    }

    ... // 忽略其他代码
}
```

如此即可完成按钮的基础平台实现，也就是在 MAUI 的界面上创建一个按钮，就会自动创建一个 WPF 对应的按钮。在 MAUI 的按钮上设置文本，将会自动同步到 WPF 的按钮，自动给 WPF 的按钮设置上文本。点击 WPF 的按钮，就会触发回 MAUI 的按钮的点击

看到这里大家也能感受到这个工作量有庞大了吧

这还没结束，以上只是介绍了使用 WPF 作为 MAUI 的底层框架如何实现 MAUI 的按钮处理器。而作为本文的核心逻辑，如何将 MAUI 的界面嵌入到 WPF 的控件里面还没介绍

其实在了解了 MAUI 的各个控件的处理器注入机制之后，就能想到如何实现将 MAUI 的界面嵌入到 WPF 的控件里面。只需要让 MAUI 的某个控件的实现对应的平台层，也就是在这里的 WPF 层，让这个平台层实现的控件加入到 WPF 的某个控件里面即可。由于 MAUI 的底层实现全部都是由 WPF 层实现的，自然也就不会存在空域等问题了

以下是 MauiForWpfHostHelper 的 HostMainPage 方法，在这个方法里面将 Maui 的 MainPage 嵌入到传入的控件里面

```csharp
    public static void HostMainPage(Panel panel, MauiApplicationProxy applicationProxy)
    {
        var application = applicationProxy.Application;
        var context = application.Handler!.MauiContext!;

        var mauiContext = new FooPanelMauiContext(panel, context);
        var mauiWindow = (Microsoft.Maui.Controls.Window) application.CreateWindow(new ActivationState(mauiContext));

        var mainPage = new MainPage()
        {
            WidthRequest = panel.Width,
            HeightRequest = panel.Height,
        };
        var platform = mainPage.ToPlatform(mauiContext);
        _ = platform;

        mainPage.Measure(panel.Width, panel.Height);

        mainPage.Layout(new Rect(0, 0, panel.Width, panel.Height));

        mauiWindow.Page = mainPage;
    }
```

将 WPF 的 Panel 容器放入到 FooPanelMauiContext 里面，然后调用 Maui 的 IApplication 创建窗口，在创建的时候自然就注入了上下文。如何将 MAUI 的 MainPage 嵌入到传入的 WPF 的 Panel 容器里的核心科技就在于注入的上下文的使用方里面

在 FooPageHandler 里面，也就是对应 MAUI 的 Page 的平台实现里面，将会实现对 MAUI 的 Page 的内容的处理，实现方式就是获取 MAUI 的平台实现控件，将平台实现控件放入到上述传入的 Panel 里面，从而让 MAUI 的控件嵌入到 WPF 控件里，具体实现就是在 FooPageHandler 实现 Content 属性的处理

```csharp
class FooPageHandler : PageHandler
{
    public FooPageHandler() : base(new PropertyMapper<IContentView, IPageHandler>(PageHandler.Mapper)
    {
        [nameof(IContentView.Content)] = MapFooContent
    })
    {
    }

    ... // 忽略其他代码

    static void MapFooContent(IContentViewHandler handler, IContentView page)
    {
        var panel = (Panel) handler.PlatformView;
        var platform = (UIElement?) page.PresentedContent?.ToPlatform(new FooTreeMauiContext(panel, handler.MauiContext!));
        panel.Children.Clear();
        if (platform != null)
        {
            panel.Children.Add(platform);
        }
    }
}
```

通过这样的方法即可让 MAUI 嵌入到 WPF 里面，而且由于采用 WPF 作为 MAUI 的底层实现，自然就没有空域问题

最后需要说明的是这样的方法只是完成了技术可行性的测试而已，远远还没有达到在具体项目可用的阶段，需要进一步的开发才能使用

当前的 MAUI 和 WPF 都是完全开源的，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。欢迎大家参与框架开发




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。