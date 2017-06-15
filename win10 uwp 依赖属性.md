# win10 uwp 依赖属性

<!--more-->

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

参见：https://stackoverflow.com/questions/4764916/listen-to-changes-of-dependency-property


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  