
# win10 uwp 如何使用DataTemplate

这是数据模板，一般用在数组的绑定，显示数组中的元素。
假如我们有一个列表，列表里是书，包括书名、作者、还有出版，那么我们只有源信息，如何把它显示到我们的ListView，就需要DataTemplate。
使用很简单，我们可以定义在资源，也可以定义在ItemTemplate。
数据模板有绑定的问题。
我们使用`Binding`和WPF其实没有多少不同，在Mode只有`OneWay`,`OneTime`,`TwoWay`。我们使用的`x:bind`在DataTemplate才和原来有一些不同。
我们使用`x:bind`需要我们对我们数据的类型，这个在前没有，我开始不知，弄了好久，最后才知道，还有一个，UWP默认是OneTime，也就是绑定只有一次。

<!--more-->


<!-- CreateTime:2019/7/29 10:02:32 -->


<div id="toc"></div>


## 转换

有时候我们绑定的类型和显示不同，例如我们绑定了一个`bool?`但是我们在 ViewModel 的类型是 bool，那么我们就需要用转换器。转换器就是继承 IValueConverter 的一个类。

UWP的 Convert 和 WPF 差不多。

数据转换一个简单方法是另外在 ViewModel 写一个属性，这个属性用于转换变量，然后在前台绑定，但是这样做不好，于是我们比较好的一个做法是做转换器，转换器是一个类，我们需要实现它才能使用，在我们常用的做法是把它写staticResource

首先是创建一个类，这个类继承IValueConverter，于是就有两个方法，我们要实现两个方法，一个是从数据源转换到 xaml ，一个是反过来。

```csharp
 public object Convert(object value, Type targetType,
            object parameter,
            string language);

 public object ConvertBack(object value, 
            Type targetType, 
            object parameter, string language);

```

一般我们实现第一个就好，最简单的实现是直接转换。假如我们是需要把`bool？`转bool，那么一个简单方法是：

```csharp
        public object Convert(object value, Type targetType,
            object parameter,
            string language)
        {
            if (value is bool?)
            {
                bool? temp = value as bool?;
                if (temp == true)
                {
                    return true;
                }
                return false;
            }
            return false;
        }

```

在我的[变大数字颜色按钮](https://github.com/lindexi/UWP/tree/master/uwp/control/RountGradualFigure) 代码在 https://github.com/lindexi/UWP/tree/master/uwp/control/RountGradualFigure 有用到转换，是把数字转颜色

我们要使用写的转换器，就需要在 xaml 写静态资源，我们也可以把他放在 viewModel，但是我们先说下放在 xaml 的。

在资源，如果是 Page 的 xaml，那么就写在`Page.Resources`，如果只是这个转换器用在一个Grid，就写在Grid，我先用Page做例子。当然写为 `FrameworkElement.Resources` 就几乎在所有控件都可以使用。

```xml
    <Page.Resources>

    </Page.Resources>

```

我的转换器名称是：ConvertBooleanNull

假如我们放在 Model 里，命名空间是 `项目.Model`，我们需要先在 xmlns 写`xmlns:view="using:项目.Model"`，view 就是一个变量，这个可以改为你需要的。于是在需要使用的地方就可以使用 view 表示所在空间。

然后在静态资源使用下面代码`<view:ConvertBooleanNull x:Key="ConvertBooleanNull">   </view:ConvertBooleanNull>`

所有的代码请看下面

```xml
    <Page.Resources>
        <view:ConvertBooleanNull x:Key="ConvertBooleanNull">   </view:ConvertBooleanNull>
    </Page.Resources>

```

在需要使用的控件，假如我们控件绑定是`x:bind`，那么在Converter需要`Converter={StaticResource ConvertBooleanNull}`

假如我们控件绑定的是 ViewModel 的 JiuYouImageShack，需要进行转换，就可以写

```xml
 <TextBox Text="{x:Bind View.JiuYouImageShack,Mode=OneWay,Converter={StaticResource ConvertBooleanNull}}"></TextBox>


```

需要知道，`x:bind`默认是 OneTime 而 `Binding` 默认是 OneWay 

参见：http://www.cnblogs.com/horan/archive/2012/02/27/2368262.html

[[UWP]了解IValueConverter - dino.c - 博客园](http://www.cnblogs.com/dino623/p/IValueConverter.html )

## 绑定Event到Command

如果希望绑定事件，可以使用 下面代码

```xml
<ListView>
    <Interactivity:Interaction.Behaviors>
         <Core:EventTriggerBehavior EventName="SelectionChanged">
                   <Core:InvokeCommandAction Command="{Binding ShowDialog}" CommandParameter="{Binding ElementName=lv,Path=SelectedItem,Converter={StaticResource converter}}"/>
         </Core:EventTriggerBehavior>
    </Interactivity:Interaction.Behaviors>
</ListView>

```


## 绑定 ObservableCollection

如果绑定的 ItemSource 是一般的 List ，那么在 List 内容改变无法看到，界面修改

需要修改 List 内容，修改页面，添加一个新的 item 可以在页面添加一个 项，不是在初始的时候进行修改，可以使用方法：

1. 在修改之后 使用 listView.Itemsource=list 的方法，重新给 Itemsource ，这是不推荐的，因为之后可能在代码写很多添加或移除，于是都需要写这个。

1. 绑定的 List 改 ObservableCollection ，这样就可以在绑定内容修改时修改 界面。

第二个是推荐的，在使用 列表，经常使用的是 ObservableCollection ，注意，他是一个泛型，必须添加类型。

但是有时候可能关心他是如何做的，关心的原因：没有实现 AddRange，也就是 ObservableCollection 对一次添加多个项比较难，需要一个一个来。

对于定义控件，可能也需要，如何绑定一个 List 可以知道已经修改。

其实 ObservableCollection 继承 INotifyCollectionChanged ，于是可以获得列表修改，一旦自己定义继承 INotifyCollectionChanged 列表，可以做到和 ObservableCollection 差不多的样子。

先把东西分来说：一个是如何定义一个和 ObservableCollection 差不多，可以绑定界面，修改就自动让界面修改。一个是如何定义控件，可以获得列表改变。

先说第一个，其中只需要定义的列表 INotifyCollectionChanged 就可以让界面跟着修改，如果自己写的没有修改，那么是自己写错了，看起来 INotifyCollectionChanged 实现不是很简单。

第二个，可以使用依赖属性，在获得值判断 e.NewValue 是 INotifyCollectionChanged ，获得 CollectionChanged 的添加新项就可以。

参见：[win10 uwp 通知列表](https://blog.lindexi.com/post/win10-uwp-%E9%80%9A%E7%9F%A5%E5%88%97%E8%A1%A8.html)

## DataTemplate 绑定 ViewModel

假如有一个 ViewModel 他有一个列表和字段

```csharp
   public List<string> Foo { set; get; } = new List<string>() { "1" };

    public string Name { get; set; } = "lindex";
```

那么在页面写一个列表

```csharp
<Grid x:Name="Grid">      

    <ListView ItemsSource="{Binding Foo}">
        <ListView.ItemTemplate>
            <DataTemplate >
                <TextBlock Text="{Binding}"></TextBlock>
            </DataTemplate>
        </ListView.ItemTemplate>
    </ListView>
</Grid>
```

可以看到页面显示一个元素，但是如何想让 TextBlock 绑定 Name 怎么做？

因为 Grid 的数据绑定 ViewModel，所以在 WPF 可以使用 `Binding RelativeSource={RelativeSource AncestorType={x:Type Grid}` 的写法绑定到指定的元素，所以获得数据，但是 UWP 不能这样写，可以使用下面的代码

```csharp
  <ListView ItemsSource="{Binding Foo}">
        <ListView.ItemTemplate>
            <DataTemplate >
                <TextBlock Text="{Binding ElementName=Grid,Path=DataContext.Name}"></TextBlock>
            </DataTemplate>
        </ListView.ItemTemplate>
    </ListView>
```

这样就可以绑定 ViewModel ，所以就可以使用属性

[https://stackoverflow.com/a/47957417/6116637](https://stackoverflow.com/a/47957417/6116637 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。