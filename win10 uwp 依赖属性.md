# win10 uwp 依赖属性

本文告诉大家如何使用依赖属性，包括在 UWP 和 WPF 如何使用。

本文不会告诉大家依赖属性的好处，只是简单告诉大家如何使用。

<!--more-->
<!-- CreateTime:2018/8/10 19:17:19 -->


<div id="toc"></div>

在 UWP 和 wpf ，如果需要创建自己的依赖属性，可以使用代码片，在 VisualStudio 可以使用 `propdp `输入两个 tab 就可以输入依赖属性。

本文最后提供修改的代码片，可以解决变量名修改出现的界面绑定不刷新。


## UWP

### 什么时候可以获取继承的依赖属性

依赖属性一般是不在构造函数写获取继承的属性的值，因为一般这时拿到的值都是没有继承，请看下面的代码

创建一个用户控件 LuenqxuhkRrjbzcf ，在他的构造函数和加载完成事件添加获得 DataContext 的值

```csharp

  构造： var t = DataContext;

          private void LuenqxuhkRrjbzcf_Loaded(object sender, RoutedEventArgs e)
        {
            var t = DataContext;
        }

```

然后把他加入到其他页面，这个页面设置了 DataContext ，但是运行在构造的断点可以看到拿到的值是空

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017%25E5%25B9%25B411%25E6%259C%258810%25E6%2597%25A5%2520111233392018114151411.jpg)

但是可以在加载完成函数拿到

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017%25E5%25B9%25B411%25E6%259C%258810%25E6%2597%25A5%2520111233392018114151439.jpg)

那么是在什么时候才可以拿到依赖属性的值？

依赖属性需要在加逻辑树才可以拿到值，所以在加入逻辑树之后，构造函数是类创建，所以这时不能拿依赖属性的值。

### 自定义可继承依赖属性

我找了很久，发现 uwp 不支持 FrameworkPropertyMetadata 所以无法自己定义可以继承的依赖属性

## WPF 

### 后台绑定 依赖属性

后台绑定 依赖属性可以使用 Binding


```csharp
            Binding bind = new Binding("绑定路径，就是哪个属性")
            {
                Source = 绑定源，如果没有设置，可以使用 DataContext,
                Mode = BindingMode.OneWayToSource
            };
           一个继承依赖的类.SetBinding(xx.xProperty, bind);
```
例如绑定 ViewModel 的 Padding  到 一个 G控件的 Padding， 可以使用下面代码


```csharp
                Binding bind = new Binding("Padding")
            {
                Source = ViewModel,
                Mode = BindingMode.OneWayToSource
            };
            G.SetBinding(Border.PaddingProperty, bind);
```

但我的问题是，绑定只能在包含 G 的类使用？

也就是在 MainPage 写了 G 这个 类，于是绑定只能写在 MainPage 类？

实际我用了一个类来测试

我把上面的绑定代码写到 ViewModel ，发现还是可以使用。

那么问题2，如果我的 ViewModel 的绑定属性是私有的，那么把绑定写在ViewModel 里，那么是否可以访问，可以看到，如果写在ViewModel 的Binding ，那么这个 Binding 是可以访问 ViewModel 的属性，虽然这个属性是私有的。但是实际绑定需要获取的不是在创建的时候拿到，所以这时是获取不到ViewModel 里的属性。

我尝试下面的代码，把 Padding 设置为 private ，然后在 ViewModel 里绑定到他，结果发现无法从绑定获得。


```csharp
       public partial class MainWindow : Window
    {
        public MainWindow()
        {
            ViewModel = new ViewModel();
            InitializeComponent();
            DataContext = ViewModel;

            ViewModel.Board = G;

            //Binding bind = new Binding("Padding")
            //{
            //    Source = ViewModel,
            //    Mode = BindingMode.OneWayToSource
            //};
            //G.SetBinding(Border.PaddingProperty, bind);
            ViewModel.Click();
        }



        public ViewModel ViewModel { get; set; }

        private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            G.Padding = new Thickness(G.Padding.Left + 1, G.Padding.Top + 1, G.Padding.Right, G.Padding.Bottom);
            Console.WriteLine(ViewModel.Pad());
        }
    }

    public class ViewModel
    {
        public Thickness Pad()
        {
            return Padding;
        }

        private Thickness Padding { get; set; }

        public Border Board { get; set; }

        public void Click()
        {
            Binding bind = new Binding("Padding")
            {
                Source = this,
                Mode = BindingMode.OneWayToSource
            };
            Board.SetBinding(Border.PaddingProperty, bind);
        }
    }
```

如果在绑定之前，设置 G 的 Padding 为一个值，那么在设置绑定之后，这个值就会被设置默认值。

如果在绑定之前，设置 G 的 Padding  为20 ，那么设置绑定之后， G 的 Padding = 0

如果需要保留这个值，可以使用临时变量。

绑定还有另一个问题，一个属性只能做一次绑定。

假如我有多个属性，把这多个属性绑定在 G 的 Padding ，那么只有最后的一个绑定可以使用，其他的绑定无法使用。


```csharp
            public Thickness BoardPadding { get; set; }
       
       public void Click()
        {
            Binding bind = new Binding("Padding")
            {
                Source = this,
                Mode = BindingMode.OneWayToSource
            };

            BindingOperations.SetBinding(Board, Border.PaddingProperty, bind);


            bind = new Binding("BoardPadding")
            {
                Source = this,
                Mode = BindingMode.OneWayToSource
            };
            Board.SetBinding(Border.PaddingProperty, bind);
        }
```

可以看到，这时 Padding 的值一直没有。

### WPF 获得依赖属性值更新

如果需要获得 G 的 Padding 的值更改，WPF 获得依赖属性 值更改可以使用下面代码


```csharp
                DependencyPropertyDescriptor.FromProperty(Border.PaddingProperty,typeof(Border)).AddValueChanged(Board,
                (s, e) =>
                {
                    Padding = Board.Padding;
                    BoardPadding = Board.Padding;
                });
```

这个方法就是获得属性的值更改

但是这个方法会出现内存泄露，可以使用 RemoveValueChanged 清除，为了使用清除，需要写一个函数。

不需要担心清除一个不存在的委托，一般在使用 AddValueChanged 之前都使用 RemoveValueChanged 清除

参见：https://stackoverflow.com/questions/4764916/listen-to-changes-of-dependency-property


### 初始化出现默认值类型与属性类型不同

定义的依赖属性是需要默认值类型和定义的一样，在一般的代码，可以使用隐式转换，但是在定义不可以使用。

例如使用类型是 double 实际给的是 int ，就会在运行出现`ArgumentException`

```csharp
        public static readonly DependencyProperty FooProperty = DependencyProperty.Register(
            "Foo", typeof(double), typeof(MainWindow), new PropertyMetadata(2));

        public double Foo
        {
            get { return (double) GetValue(FooProperty); }
            set { SetValue(FooProperty, value); }
        }
```

虽然定义`double a=2;`是对的，但是在这里定义的 `2`默认是错误的，需要写`2d`才是对的

### 修改属性名称

默认的代码片生成代码的属性名称是字符串，但是字符串有个缺点，如果修改了变量名，那么界面绑定就无法找到。

建议把字符串换为C# 6.0 带来的新特性

```csharp
       public static readonly DependencyProperty FooProperty = DependencyProperty.Register(
            nameof(Foo), typeof(double), typeof(MainWindow), new PropertyMetadata(2d));

        public double Foo
        {
            get { return (double) GetValue(FooProperty); }
            set { SetValue(FooProperty, value); }
        }
```

通过修改代码片就可以做到，如何修改请看 [resharper 自定义代码片](http://lindexi.oschina.io/lindexi//post/resharper-%E8%87%AA%E5%AE%9A%E4%B9%89%E4%BB%A3%E7%A0%81%E7%89%87/ )

下面就是修改后的代码

```csharp
public static readonly $dependencyProperty$ $propertyName$Property = $dependencyProperty$.Register(
  nameof($propertyName$), typeof($propertyType$), typeof($containingType$), new PropertyMetadata(default($propertyType$)));

public $propertyType$ $propertyName$
{
  get { return ($propertyType$) GetValue($propertyName$Property); }
  set { SetValue($propertyName$Property, value); }
```

可以直接粘贴进去Resharper的代码

或者导入我的设置，点击[下载](http://image.acmx.xyz/%E4%BE%9D%E8%B5%96%E5%B1%9E%E6%80%A71685E00E-67E5-4343-A467-84862A1EE502.DotSettings)

如果想要使用的是 C# 7 的特性，可以修改代码片，或者点击[下载](http://image.acmx.xyz/%E4%BE%9D%E8%B5%96%E5%B1%9E%E6%80%A72E6789E0-E16E-4B2F-896B-671CC1F21B11.DotSettings)导入

```csharp
public static readonly $dependencyProperty$ $propertyName$Property = $dependencyProperty$.Register(
  nameof($propertyName$), typeof($propertyType$), typeof($containingType$), new PropertyMetadata(default($propertyType$)));

public $propertyType$ $propertyName$
{
  get => ($propertyType$) GetValue($propertyName$Property);
  set => SetValue($propertyName$Property, value);
}
```

![](https://i.loli.net/2018/07/29/5b5d61367fc0c.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  