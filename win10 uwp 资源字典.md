# win10 uwp 资源字典

本文告诉大家如何定义、使用资源

<!--more-->
<!-- csdn -->

本文主要翻译[ResourceDictionary and XAML resource references - UWP app developer ](https://docs.microsoft.com/zh-cn/windows/uwp/design/controls-and-patterns/resourcedictionary-and-xaml-resource-references )，里面的代码我重新写了一下，有一些不相同。

一般资源在 xaml 定义，定义的地方可以是在 Page ，请看下面的代码

```csharp
<Page
    x:Class="KrahfcjjqKzz.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">

    <Page.Resources>
        <x:String x:Key="TalsdtiiKjsvk">林德熙</x:String>
        <x:String x:Key="KsjdqKoqij">csdn</x:String>
    </Page.Resources>

    <TextBlock Text="{StaticResource TalsdtiiKjsvk}" Foreground="Gray" VerticalAlignment="Center"/>
</Page>
```

可以看到，上面的代码在 Page 的资源定义了两个字符串，然后在控件使用了定义的资源。实际可以在 Resources 定义几乎任意的资源，但是要求这些资源有默认构造函数，而且支持定义为资源。例如支持共享的类型，styles、templates、brushes，在下面会告诉大家具体哪些元素是可以共享。

使用资源的方法是在需要使用的地方使用 StaticResource 获得。如果需要从后台拿到资源，请看[后台获取资源 ](https://lindexi.gitee.io/post/win10-uwp-%E5%90%8E%E5%8F%B0%E8%8E%B7%E5%8F%96%E8%B5%84%E6%BA%90.html )

而 StaticResource 获得资源是通过一个特殊的寻找方法，这个方法在后面告诉大家。

## 资源的key

从上面的代码可以看到，所有的资源定义都有一个 Key ，通过这个 Key 就可以让 StaticResource 找到需要的资源。但是存在一些特殊的资源是可以不使用 Key 的，下面让我来告诉大家有哪些东西可以不添加 key 

### Style 

对于 Style 和 ControlTemplate 等，具有TargetType表示这是属于哪个类型的 样式，如果不定义 Key ，那么在这个资源定义包起来的控件都会使用这个样式，请看下面的代码

```csharp
<Page
    x:Class="KrahfcjjqKzz.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">

    <Page.Resources>
        <x:String x:Key="TalsdtiiKjsvk">林德熙</x:String>
        <x:String x:Key="KsjdqKoqij">csdn</x:String>
        <Style TargetType = "TextBlock">
            <Setter Property="Margin" Value="10,10,10,10"/>
        </Style>
    </Page.Resources>

    <TextBlock Text="{StaticResource TalsdtiiKjsvk}" Foreground="Gray" VerticalAlignment="Center"/>
</Page>
```

可以看到，没有设置 TextBlock 的 Style ，但是自动就修改了 TextBlock 的样式

### DataTemplate

对于 DataTemplate 也可以不给 Key ，因为一般的 DataTemplate 都会指定数据类型，所以对于没有指定 Key 的 DataTemplate 会自动用在他使用的数据类型

不过不建议使用这个方法

### Name

对于已经定义了命名的资源可以不使用Key ，因为通过命名可以可以拿到资源。在资源定义 Name 是 UWP 才有的，在 WPF 是不能这样做，定义了 Name 可以很快在后台代码拿到资源，但是运行效率 Name 会比 Key 低，因为在页面 Loaded 之后需要初始化这个资源。

## 所有的元素都可以定义资源

实际上不只是页面可以添加资源，对所有的 FrameworkElement 都可以定义资源。如果大家还不知道什么是 FrameworkElement ，那么简单可以说，所有显示在界面的元素都是 FrameworkElement ，所以所有显示的元素都可以定义资源。包括面板和自定义控件。

而且资源的寻找居然靠近优先，也就是在页面定义的资源和在元素定义的资源，会先在元素找，如果元素可以找到资源，就不会在页面找

```csharp
<Page
    x:Class="TobHrv.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">

    <Page.Resources>
        <x:String x:Key="Lindexi">林德熙</x:String>
    </Page.Resources>

    <Border>
        <Border.Resources>
            <x:String x:Key="Lindexi">逗比开发者</x:String>
        </Border.Resources>
        <TextBlock Text="{StaticResource Lindexi}" Foreground="Gray" VerticalAlignment="Center"/>
    </Border>
</Page>
```

可以看到这个软件运行显示的是 逗比开发者而不是 林德熙，因为相同的 Key 定义在元素，资源先在元素找，找到了就不会去页面找。资源寻找的方向是 TextBlock -> Border -> Page ，因为在 Boarder 就找到资源，所以在页面的资源就不会找了。通过这个方法可以自定义需要的资源，也就是在 App.xaml 定义一般使用的资源，然后在 Page 定义页面的资源，在元素定义特殊资源。

但是需要知道，如果使用的是 x:Bind ，那么只会在页面找，不会在元素找。这是很重要的，具体请看[x:Bind 无法获得资源](https://lindexi.gitee.io/post/win10-uwp-xBind-%E6%97%A0%E6%B3%95%E8%8E%B7%E5%BE%97%E8%B5%84%E6%BA%90.html )

## 合并资源字典

从上面的代码实际还是看不出资源存在的问题，实际上的资源需要的代码是比较多的，特别是特殊的 Style ，一个 Style 一般有很多行，如果都写在页面，那么需要看到的代码很多。所以建议的方法是把资源写在一个文件，这个文件就是资源文件。把资源写在文件可以让资源在多个项目使用，也可以在需要使用资源的项目使用，在不需要使用资源的项目就不添加。因为资源的创建也需要内存。

下面创建一个资源字典 SedpwbvkKbrjlpi.xaml ，在里面定义一个资源

```csharp
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" 
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:SqdSgjd">

    <SolidColorBrush x:Key="brush" Color="Red"/>

</ResourceDictionary>
```

在需要使用资源的地方可以用下面的代码引用这个资源

```csharp
<Page
    x:Class="SqdSgjd.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Page.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="SedpwbvkKbrjlpi.xaml"/>
            </ResourceDictionary.MergedDictionaries>

        </ResourceDictionary>
    </Page.Resources>

    <TextBlock Foreground="{StaticResource brush}" Text="林德熙" VerticalAlignment="Center"/>
</Page>
```

需要知道上面的代码存在两个问题，一个是资源的路径，需要把资源写为相对文件的路径，如果需要写绝对，那么请使用 `ms-appx`的方法。另外，对于资源的命名，都是用 Aa 的命名方式，而不是开头小写。

如果创建了另一个资源字典 KlgnkTbyt.xaml ，使用下面的代码可以引用这个字典

```csharp
<Page
    x:Class="SqdSgjd.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Page.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="SedpwbvkKbrjlpi.xaml"/>
                <ResourceDictionary Source="KlgnkTbyt.xaml"/>
            </ResourceDictionary.MergedDictionaries>

        </ResourceDictionary>
    </Page.Resources>

    <TextBlock Foreground="{StaticResource brush}" Text="林德熙" VerticalAlignment="Center"/>
</Page>
```

但是如果在 KlgnkTbyt.xaml 也定义了一个在第一个字典也存在的 Key ？ 会找到什么？实际上资源可以被重新定义，在后面的定义会覆盖前面的，所以如果有两个从重复定义，会使用后面一个。

