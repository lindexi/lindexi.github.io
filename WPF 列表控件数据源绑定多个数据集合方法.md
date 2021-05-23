# WPF 列表控件数据源绑定多个数据集合方法

在 WPF 用的多的列表控件如 ListBox 或 ListView 等，本文告诉大家在这些列表控件上进行绑定多个数据集合来源的多个实现方法。如有一个显示动物列表的控件，需要绑定的数据来源是阿猫和阿狗两个 ObservableCollection 列表，不在后台代码编写合并集合的代码情况下，可以通过 XAML 的编写，绑定多个数据集合

<!--more-->
<!-- CreateTime:2021/5/22 9:27:42 -->


<!-- 发布 -->

## 准备

在开始之前，咱先搭建一点测试使用的代码，假定咱有一个 列表控件 准备绑定到的数据源是两个 ObservableCollection 对象，下面来定义这两个 ObservableCollection 对象和对应的 阿猫和阿狗 的代码

```csharp
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();

            for (int i = 0; i < 10; i++)
            {
                Dogs.Add(new Dog()
                {
                    Name = "Dog" + i
                });

                Cats.Add(new Cat()
                {
                    Name = "Cat" + i
                });
            }

            DataContext = this;
        }

        public ObservableCollection<Dog> Dogs { get; } = new ObservableCollection<Dog>();
        public ObservableCollection<Cat> Cats { get; } = new ObservableCollection<Cat>();
    }

    public class Dog : Animal
    {
    }

    public class Cat : Animal
    {
    }

    public class Animal
    {
        public string Name { get; set; }
    }
```

可以看到以上代码里面存在两个 ObservableCollection 对象，同时 MainWindow 的 DataContext 就是 MainWindow 对象。咱需要将两个 ObservableCollection 对象作为数据源，放在相同的一个 ListBox 里面

下面是多个不同的实现方式，解决如何在 WPF 中在 ListBox 或 ListView 绑定多个数据集合 ObservableCollection 对象

## 通过 CollectionViewSource 方式

在 ListView 或 ListBox 资源里面，添加 CollectionViewSource 绑定到集合里面，然后在 ItemsSource 使用 CompositeCollection 进行绑定，代码如下

```xml
    <ListBox>
      <ListBox.Resources>
        <CollectionViewSource x:Key="DogCollection" Source="{Binding Dogs}"/>
        <CollectionViewSource x:Key="CatCollection" Source="{Binding Cats}"/>
      </ListBox.Resources>
      <ListBox.ItemsSource>
        <CompositeCollection>
          <CollectionContainer Collection="{Binding Source={StaticResource DogCollection}}"/>
          <CollectionContainer Collection="{Binding Source={StaticResource CatCollection}}"/>
        </CompositeCollection>
      </ListBox.ItemsSource>

      <ListBox.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding Name}"></TextBlock>
        </DataTemplate>
      </ListBox.ItemTemplate>

    </ListBox>
```

这个方法的优势在于可以完全使用 XAML 编写内容，但是缺点在于有重复的代码，如有多个绑定的集合对象，就需要在资源和 CompositeCollection 里面定义多个 CollectionViewSource 和 CollectionContainer 对象

如果绑定的集合数量不多，那么此写法还成，但如果集合数量比较多，而且需要不断变更顺序，那以上写法就有坑

此方法请参考 [WPF 很少人知道的科技 - walterlv](https://blog.walterlv.com/post/those-people-dont-know-about-wpf.html )


## 通过 CompositeCollection 动态绑定

在 ListView 或 ListBox 的资源里面定义了 CompositeCollection 通过控件的 DataContext 绑定多个集合，代码如下

```xml
        <CompositeCollection x:Key="MyColl">
          <CollectionContainer Collection="{Binding DataContext.Dogs, Source={x:Reference MyList}}"/>
          <CollectionContainer Collection="{Binding DataContext.Cats, Source={x:Reference MyList}}"/>
        </CompositeCollection>
```

以上代码的 MyList 就是集合控件，此方法需要用到 `x:Reference` 获取对象的引用，同时需要通过 `DataContext` 的某个属性获取到对应的属性，全部代码如下

```xml
    <ListBox x:Name="MyList" ItemsSource="{DynamicResource MyColl}">
      <ListBox.Resources>
        <CompositeCollection x:Key="MyColl">
          <CollectionContainer Collection="{Binding DataContext.Dogs, Source={x:Reference MyList}}"/>
          <CollectionContainer Collection="{Binding DataContext.Cats, Source={x:Reference MyList}}"/>
        </CompositeCollection>
      </ListBox.Resources>
      <ListBox.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding Name}"></TextBlock>
        </DataTemplate>
      </ListBox.ItemTemplate>
    </ListBox>
```

对比上面的方法，此方法可以让绑定集合的代码只写一次，看起来代码更少一点。但不足的地方在于绑定 ItemsSource 需要用到 DynamicResource 的方式，相对性能不如上面方法。为什么需要 DynamicResource 资源？原因是资源本身定义在 Resources 里面。为什么资源需要定义在控件里面的 Resource 里面？原因是为了获取到控件的 `x:Reference` 对象。也就是说需要在控件创建出来之后，才能通过 `x:Reference` 获取控件，而控件的数据内容需要依赖资源的定义，因此也只有以上方式的写法


如果能从控件的上层容器拿到数据对象，那可以将资源定义在容器里面，通过 StaticResource 绑定到静态资源。如放在 Window 的 Resources 里

```xml
<Window x:Class="CibairyafocairluYerkinemde.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:CibairyafocairluYerkinemde"
        mc:Ignorable="d"
        x:Name="Root"
        Title="MainWindow" Height="450" Width="800">
  <Window.Resources>
    <CompositeCollection x:Key="MyColl">
      <CollectionContainer Collection="{Binding DataContext.Dogs, Source={x:Reference Root}}"/>
      <CollectionContainer Collection="{Binding DataContext.Cats, Source={x:Reference Root}}"/>
    </CompositeCollection>
  </Window.Resources>

  <Grid>
   <ListBox x:Name="MyList" ItemsSource="{StaticResource MyColl}" >
      <ListBox.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding Name}"></TextBlock>
        </DataTemplate>
      </ListBox.ItemTemplate>
    </ListBox>
  </Grid>
</Window>
```

以上写法没有啥缺点，也不存在动态资源的性能问题。但实际上在有动态资源下，性能问题也是很小的问题，对比渲染控件本身，动态绑定性能可以忽略

## 通过多绑定方法

此方法需要添加一点后台代码，定义 CompositeCollectionConverter 转换器，实现逻辑是通过多绑定的方法，将多个数据集合当成多个参数进行绑定

```csharp
    <ListBox>
      <ListBox.ItemsSource>
        <MultiBinding Converter="{x:Static local:CompositeCollectionConverter.Default}">
          <Binding Path="Dogs" />
          <Binding Path="Cats" />
        </MultiBinding>
      </ListBox.ItemsSource>
      <ListBox.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding Name}"></TextBlock>
        </DataTemplate>
      </ListBox.ItemTemplate>
    </ListBox>
```

可以看到此方法的 XAML 代码量最小，只是需要一个辅助的 CompositeCollectionConverter 类，代码如下

```csharp
    public class CompositeCollectionConverter : IMultiValueConverter
    {
        public static readonly CompositeCollectionConverter Default = new CompositeCollectionConverter();

        public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
        {
            var compositeCollection = new CompositeCollection();
            foreach (var value in values)
            {
                if (value is IEnumerable enumerable)
                {
                    compositeCollection.Add(new CollectionContainer { Collection = enumerable });
                }
                else
                {
                    compositeCollection.Add(value);
                }
            }

            return compositeCollection;
        }

        public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
        {
            throw new NotSupportedException("CompositeCollectionConverter ony supports oneway bindings");
        }
    }
```

可以将 CompositeCollectionConverter 放在库里面，这样就可以让 XAML 代码看起来简单

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/17789b4bd8e121e2469cfc534094903994cbab81/CibairyafocairluYerkinemde) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/17789b4bd8e121e2469cfc534094903994cbab81/CibairyafocairluYerkinemde) 欢迎小伙伴访问

## 参考

本文以上方法参考了如下博客

[c# - CompositeCollection + CollectionContainer: Bind CollectionContainer.Collection to property of ViewModel that is used as DataTemplates DataType - Stack Overflow](https://stackoverflow.com/questions/19243109/compositecollection-collectioncontainer-bind-collectioncontainer-collection-t )

[wpf - How do you bind a CollectionContainer to a collection in a view model? - Stack Overflow](https://stackoverflow.com/q/6446699/6116637 )

[WPF 很少人知道的科技 - walterlv](https://blog.walterlv.com/post/those-people-dont-know-about-wpf.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、 使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
