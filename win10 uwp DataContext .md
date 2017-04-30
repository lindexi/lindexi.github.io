# win10 uwp DataContext 

本文告诉大家DataContext的多种绑法。

适合于WPF的绑定和UWP的绑定。

<!--more-->
<!-- csdn -->

最简单的绑定是写在资源。


```csharp
    <Page.Resources>
        <viewModel:ViewModel x:Key="ViewModel"></viewModel:ViewModel>
    </Page.Resources>
```
这时就可以在Grid绑定


```xml
     <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"
          DataContext="{StaticResource ViewModel}">

    </Grid>
```

因为很多WPF程序都是放在 Window 不是放在页，所以为了在UWP和WPF使用的都是相同，可以用FrameworkElement代替 Page 于是在页面任何地方都可以放。

可以这个方法有个缺点，无法在页面使用 DataContext 绑定，只能在 资源后面的 Grid 使用。因为资源是有顺序，Page 在资源之前，于是 Page 就无法绑定。在WPF的也一样。

如果只有一个页面，而且使用的地方也是在 页面的内容，那么建议使用这个方法。

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201743091652.jpg)

另一个方法是把他写到 app


```xml
    <Application.Resources>
        <viewModel:ViewModel x:Key="ViewModel"></viewModel:ViewModel>
    </Application.Resources>
```

这样在程序任何地方都可以使用

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201743091744.jpg)

我的想法，如果是 ViewModel ，那么写在这里，对于MVVM的ViewModel ，MainPage 对应的 ViewModel 建议写在这里。

如果写在这，代码使用 `(ViewModel) App.Current.Resources["ViewModel"] ` 就可以获得，也就是在任意的代码都可以使用这个方法获得。

在程序运行时都可以得到 ViewModel ，这是这方法适合的地方。

当然缺点是，如果你写了很多个 ViewModel 在资源，在程序运行都会占内存，也不会释放，所以一般建议只写ViewModel ，不要写多个。

如果对于一个 ViewModel  只有一个页面使用，那么可以不需要写在 App ，因为这样会让其它的页面都可以访问

遇到上面的需要，只有一个页面需要 ViewModel ，可以直接写


```csharp
     <Page.DataContext>
        <vm:ViewModel></vm:ViewModel>
    </Page.DataContext>
```

这个方法可以让ViewModel和页面都在一个时间，也就是关闭了页面，也就自动关了 ViewModel ，说了这么多，好像还没说如何在代码使用 viewModel 。上面的所有方法在代码使用 ViewModel 都相同。

先定义属性 ViewModel ，然后在 构造写从 DataContext 转换。记得写最后


```csharp
        public sealed partial class MainPage : Page
    {
        public MainPage()
        {
            this.InitializeComponent();
            ViewModel = (ViewModel) DataContext;
        }

        private ViewModel ViewModel { set; get; }
    }
```

为何需要把 ViewModel 转换写在最后，我就不继续解释。

关于为何使用 cast 而不是 as ，因为已经确定了现在使用的类型就是 ViewModel ，我也需要使用的是 ViewModel 不是其他，如果有人改了其它的类型，我必须报错，于是就使用 cast ，如果使用了 Cast 那么看日志比较容易看到是那里写错。

除了在 xaml 定义DataContext，一个常用方法是在 代码定义


```csharp
            public MainPage()
        {
            ViewModel = new ViewModel();
            this.InitializeComponent();
            DataContext = ViewModel;
        }

        private ViewModel ViewModel
        {
            set; get;
        }
```

这个方法也是推荐的，可以在代码定义，但是这样在 xaml  写 binding 就不会有提示。

如果只在代码写新建 ViewModel ，不定义 DataContext ，把他写在 xaml ，那么就可以获得提示。

这里的 提示，在输入的时候，写一个变量会提示这个变量，自动给你选。没有提示容易写错代码，而且变量改名了，xaml不会随着改。


```csharp
            public MainPage()
        {
            ViewModel = new ViewModel();
            this.InitializeComponent();
        }

        private ViewModel ViewModel
        {
            set; get;
        }
```


```csharp
        DataContext="{Binding RelativeSource={RelativeSource Self},Path=ViewModel}"
```

这句代码是写在 Page ，如果写在其他的 Grid 得到就不会有 ViewModel 。

大概就是所有的可以定义 DataContext 的方法。

如果你还有新的方法，欢迎讨论。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
