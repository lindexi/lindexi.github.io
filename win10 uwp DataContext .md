# win10 uwp DataContext 

本文告诉大家DataContext的多种绑法。

适合于WPF的绑定和UWP的绑定。

我告诉大家很多个方法，所有的方法都有自己的优点和缺点，可以依靠自己喜欢的用法使用。当然，可以在新手面前秀下，一个页面一个绑定方法。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->


开始是从最简单的来说起。

## 需要知道的

### 用户控件

如果有使用用户控件，那么容易被这个坑啦，如果发现自己的绑定失败了，那么需要看一下是不是因为用户控件绑定和其他控件不相同。

先创建一个用户控件 LuenqxuhkRrjbzcf ，这是一个空白的用户控件，只需要修改背景色

```csharp
    <Grid Background="Coral">
        <TextBlock Text="lindexi.oschina.io" Margin="135,103,-135,-103"></TextBlock>
    </Grid>
```

然后在首页添加这个控件

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018114144020.jpg)

```csharp
        <local:LuenqxuhkRrjbzcf ></local:LuenqxuhkRrjbzcf>
```

这时可以看到控件显示，然后把他的 Visbilibity 绑定到 ViewModel 的属性，这时的属性的值是 Collapsed ，所以添加到首页的控件是看不到的

```csharp
    public class ViewModel
    {
        public Visibility AuynPsfqq { get; set; } = Visibility.Collapsed;
    }
```

```csharp
        <local:LuenqxuhkRrjbzcf Visibility="{Binding AuynPsfqq}"></local:LuenqxuhkRrjbzcf>

```

这时可以发现，运行看不到控件，但是如果在用户控件设置了 DataContex 那么绑定就找不到源

```csharp
        private void LuenqxuhkRrjbzcf_Loaded(object sender, RoutedEventArgs e)
        {
            DataContext = this;
        }
```

接下来添加两个按钮在首页，一个是设置用户控件的 DataContext ，一个删除，这时可以看到界面出现变化

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017%25E5%25B9%25B411%25E6%259C%258810%25E6%2597%25A5%252011123339.gif)

```csharp
<Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="203*"/>
            <RowDefinition Height="47*"/>
        </Grid.RowDefinitions>
        <local:LuenqxuhkRrjbzcf x:Name="IonMjheadyz" Visibility="{Binding AuynPsfqq}" ></local:LuenqxuhkRrjbzcf>
        <Grid Grid.Row="1">
            <Button Margin="10,0,0,0" Content="设置用户控件" Click="NefjxuqelLriklu_OnClick"></Button>
            <Button Content="删除" HorizontalAlignment="Right" Click="SfpgucmgtYserkpend_OnClick" Margin="0,27,86,88"></Button>
        </Grid>
        </Grid>
```

```csharp
        private void NefjxuqelLriklu_OnClick(object sender, RoutedEventArgs e)
        {
            IonMjheadyz.DataContext = this;
        }

        private void SfpgucmgtYserkpend_OnClick(object sender, RoutedEventArgs e)
        {
            IonMjheadyz.ClearValue(DataContextProperty);
        }
```
    
因为 DataContext 是依赖属性，如果设置依赖属性，那么就是使用自己的值，如果没有就使用上一级的值。绑定的数据就从 DataContext 拿，所以给用户控件设置 DataContext 就会让界面的绑定找不到值，所以绑定失败。

## 资源绑定

### page 资源绑定

最简单的绑定是写在资源。


```csharp
    <Page.Resources>
        <viewModel:ViewModel x:Key="ViewModel"></viewModel:ViewModel>
    </Page.Resources>
```
这时就可以在Grid绑定，当然缺点就是 后台代码无法直接使用，需要经过转换才可以使用。


```xml
     <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"
          DataContext="{StaticResource ViewModel}">

    </Grid>
```

因为很多WPF程序都是把界面放在 Window 而不是放在页，所以为了在 UWP 和WPF使用的都是相同。可以用 FrameworkElement 代替 Page 。因为所有控件几乎都继承于 FrameworkElement 于是在页面任何地方都可以放这句话，不需要多余修改。所以刚才的 `Page.Resources` 就可以修改为 `FrameworkElement.Resources`

可是这个方法有个缺点，无法在页面 Page 元素上使用 DataContext 绑定，只能在 资源后面的 Grid 使用。因为资源是有顺序，Page 在资源之前，于是 Page 就无法绑定。在WPF的也一样。提示的错误参见下图。

如果只有一个页面，而且使用的地方也是在 页面的内容，那么建议使用这个方法。

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201743091652.jpg)

如果需要在 Page 的元素也绑定到 ViewModel ，那么可以参见下面的方法。

### app 资源绑定

另一个方法是把他写到 app ，代码就是


```xml
    <Application.Resources>
        <viewModel:ViewModel x:Key="ViewModel"></viewModel:ViewModel>
    </Application.Resources>
```

这样在程序任何地方都可以使用

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201743091744.jpg)

我的想法，如果是 ViewModel ，那么写在这里，对于 MVVM 的 ViewModel ，MainPage 对应的 ViewModel 建议写在这里。

如果写在这，代码使用 `(ViewModel) App.Current.Resources["ViewModel"] ` 就可以获得，也就是在任意的代码都可以使用这个方法获得。参见：[win10 uwp 后台获取资源](http://lindexi.oschina.io/lindexi/post/win10-uwp-%E5%90%8E%E5%8F%B0%E8%8E%B7%E5%8F%96%E8%B5%84%E6%BA%90/)

这个方法的优点：
在程序运行时都可以得到 ViewModel ，这是这方法适合的地方。

当然缺点是，如果你写了很多个 ViewModel 在资源，在程序运行都会占内存，也不会释放，所以一般建议只写ViewModel ，不要写多个。

### DataContext 新建资源

如果对于一个 ViewModel  只有一个页面使用，那么可以不需要写在 App ，因为这样会让其它的页面都可以访问

遇到上面的需要，只有一个页面需要 ViewModel ，可以直接写


```csharp
     <Page.DataContext>
        <vm:ViewModel></vm:ViewModel>
    </Page.DataContext>
```

这个方法可以让ViewModel和页面都在一个时间，也就是关闭了页面，也就自动关了 ViewModel ，说了这么多，好像还没说如何在代码使用 viewModel 。上面的所有方法在代码使用 ViewModel 都相同。

### 后台代码获得资源

先定义属性 ViewModel ，然后在 构造写从 DataContext 转换。记得写构造函数的最后，在 InitializeComponent 的后面。


```csharp
        public sealed partial class MainPage : Page
    {
        public MainPage()
        {
            this.InitializeComponent();
            ViewModel = (ViewModel) DataContext; //这是 cast 方法，直接转换，不要使用 as 的方法。
        }

        private ViewModel ViewModel { set; get; }
    }
```

为何需要把 ViewModel 转换写在最后，我就不继续解释。

关于为何使用 cast 而不是 as ，因为已经确定了现在使用的类型就是 ViewModel ，我也需要使用的是 ViewModel 不是其他，如果有人改了其它的类型，我必须报错，于是就使用 cast ，如果使用了 Cast 那么看日志比较容易看到是那里写错。

## 代码定义资源

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

### 代码定义，xaml绑定

这里的 提示 指的是，在 xaml 输入的时候，写一个变量不需要完全自己写。和后台代码一样，会提示这个变量，自动给你选。没有提示容易写错代码，而且变量改名了，xaml不会随着改。


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

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
