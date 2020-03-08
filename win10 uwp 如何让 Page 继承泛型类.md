# win10 uwp 如何让 Page 继承泛型类

本文告诉大家一个方法让 Page 继承一个泛型类。

我使用自己的[框架](https://www.nuget.org/packages/lindexi.uwp.Framework)的时候，发现每个页面都需要添加一些相同的代码，所以就想把他写出来，作为一个基础的 Page 。

这个 Page 需要指定自己的 ViewModel 但是这个 ViewModel 却是每个 Page 不相同，所以一个好的方法是指定他的泛型类。但是这时如果使用 Page 继承，就会出现编译错误。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<!-- csdn -->

因为需要使用下面的代码才可以从页面拿到传入的 ViewModel ，所以所有的页面都添加相同的代码，在软件上是比较差的

```csharp
    public sealed partial class MainPage : Page
    {
        public MainPage()
        {
            this.InitializeComponent();
        }

        public ViewModel ViewModel { set; get; }

        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            ViewModel = (ViewModel) e.Parameter;
            DataContext = ViewModel;
            base.OnNavigatedTo(e);
        }
    }
```

如果可以使用泛型，那么代码会简单很多

首先写出一个基础类  DslujbefGgtvl 这个类继承 Page ，于是就可以把所有 Page 需要写的代码只写一次

```csharp
    public class DslujbefGgtvl<T> : Page where T: IViewModel
    {
        public T ViewModel { set; get; }

        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            ViewModel = (T) e.Parameter;
            DataContext = ViewModel;
            base.OnNavigatedTo(e);
        }
    }
```

这样 Page 继承 DslujbefGgtvl 就需要告诉他现在需要哪个 ViewModel 不需要其他的代码

```csharp
    public sealed partial class MainPage : DslujbefGgtvl<ViewModel>
    {
        public MainPage()
        {
            this.InitializeComponent();
        }
    }
```

但是这时会出现错误`Error CS0263: “MainPage”的分部声明一定不能指定不同的基类 (23, 33)` 这是因为 xaml 没有继承  DslujbefGgtvl

打开 xaml 把 Page 替换为 DslujbefGgtvl 然后添加 TypeArguments 

```csharp
<local:DslujbefGgtvl
    x:TypeArguments="local:ViewModel" x:Class="TwipmeHpka.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:TwipmeHpka"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">

    </Grid>
</local:DslujbefGgtvl>

```

这时编译出现    Error CS0305: 使用泛型 类型“DslujbefGgtvl<T>”需要 1 个类型参数 (1, 1) ，所以看起来 Page 无法继承 泛型类

一个解决方法是写另一个类去继承，下面我写 RavthuVythrbe 继承  DslujbefGgtvl 这样就可以使用

```csharp
   public sealed partial class MainPage : RavthuVythrbe
    {
        public MainPage()
        {
            this.InitializeComponent();
        }
    }

    public class RavthuVythrbe : DslujbefGgtvl<ViewModel>
    {
        
    }

    public class DslujbefGgtvl<T> : Page where T: IViewModel
    {
        public T ViewModel { set; get; }

        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            ViewModel = (T) e.Parameter;
            DataContext = ViewModel;
            base.OnNavigatedTo(e);
        }
    }
```

```csharp
<local:RavthuVythrbe
    x:Class="TwipmeHpka.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:TwipmeHpka"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">

    </Grid>
</local:RavthuVythrbe>
```

所以需要添加一个类来继承泛型的才可以被Page继承。

这个方法需要添加一个类，但是还有另一个方法可以写在框架，需要把 xaml 继承 Page 修改为 IPage

```csharp
    public sealed partial class MainPage : DslujbefGgtvl<ViewModel>, IPage

    public interface IPage
    {
         UIElement Content {  get;  set; }
    }

```

```csharp
<local:IPage
    x:Class="TwipmeHpka.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:TwipmeHpka"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <local:IPage.Content>
        <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
            <TextBlock Margin="10,10,10,10" Text="lindexi"></TextBlock>
        </Grid>
    </local:IPage.Content>

</local:IPage>

```

所以 page 不可以继承泛型，但是可以继承接口，所以在框架可以使用这个方法让 Page 继承一个泛型的类，这样可以减少代码

实际使用的 IPage 的代码

```csharp
   public interface IPage: IFrameworkElement
    {
        UIElement Content { get; set; }
        Frame Frame { get; }
        NavigationCacheMode NavigationCacheMode { get; set; }
        AppBar TopAppBar { get; set; }
        AppBar BottomAppBar { get; set; }
    }

    public interface IFrameworkElement
    {
        TriggerCollection Triggers { get; }
        ResourceDictionary Resources { get; set; }
        object Tag { get; set; }
        string Language { get; set; }
        double ActualWidth { get; }
        double ActualHeight { get; }
        double Width { get; set; }
        double Height { get; set; }
        double MinWidth { get; set; }
        double MaxWidth { get; set; }
        double MinHeight { get; set; }
        double MaxHeight { get; set; }
        HorizontalAlignment HorizontalAlignment { get; set; }
        VerticalAlignment VerticalAlignment { get; set; }
        Thickness Margin { get; set; }
        string Name { get; set; }
        Uri BaseUri { get; }
        object DataContext { get; set; }
        Style Style { get; set; }
        DependencyObject Parent { get; }
        FlowDirection FlowDirection { get; set; }
        event RoutedEventHandler Loaded;
        event RoutedEventHandler Unloaded;
        event SizeChangedEventHandler SizeChanged;
        event EventHandler<object> LayoutUpdated;
        object FindName(string name);
        void SetBinding(DependencyProperty dp, BindingBase binding);
    }
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  