# WPF 绑定的默认模式

小伙伴绑定了一个属性，但是发现属性在更新的时候没有同步到后台，他说在 WPF 绑定的默认值是什么？为什么没有设置 Mode 的属性，有的是双向有的是单向？本文就来告诉大家在 WPF 定义的依赖属性是如何控制绑定的是双向还是单向的方法

<!--more-->
<!-- CreateTime:2019/4/12 9:38:58 -->

<!-- csdn -->

在依赖属性或附加属性，都可以在定义的时候传入 FrameworkPropertyMetadata 请看代码

```csharp

        public static readonly DependencyProperty TwoWayProperty =
            DependencyProperty.Register("TwoWay", typeof(string), typeof(MainWindow), new FrameworkPropertyMetadata(""));
```

上面代码的使用和默认不相同，修改 PropertyMetadata 为 FrameworkPropertyMetadata 虽然传入的默认值参数都是一样的，但是 FrameworkPropertyMetadata 可以传入更多的参数，如可以传入 FrameworkPropertyMetadataOptions 变量

在 FrameworkPropertyMetadataOptions 变量可以通过设置 BindsTwoWayByDefault 指定这个值默认的绑定是双向的

虽然从 Binding 的 Mode 的枚举的定义是

```csharp
    public enum BindingMode
    {
        TwoWay,
        OneWay,
        OneTime,
        OneWayToSource,
        Default
    }
```

默认的枚举值 0 是 TwoWay 但是在 Mode 属性通过特性设置了默认的值是 Default 而如果设置默认的值是 Default 就会读取绑定的属性的对应的 FrameworkPropertyMetadata 是否有设置默认是双向

```csharp
        [DefaultValue(BindingMode.Default)]

```

在 TextBlock 这些控件，有很多属性的绑定都是双向的，但是如果是小伙伴定义的控件，他可以定义出默认是双向绑定的或没有的

```csharp
      public string TwoWay
        {
            get { return (string) GetValue(TwoWayProperty); }
            set { SetValue(TwoWayProperty, value); }
        }


        public static readonly DependencyProperty TwoWayProperty =
            DependencyProperty.Register("TwoWay", typeof(string), typeof(MainWindow), new FrameworkPropertyMetadata("", FrameworkPropertyMetadataOptions.BindsTwoWayByDefault));

        public string OneWay
        {
            get { return (string) GetValue(OneWayProperty); }
            set { SetValue(OneWayProperty, value); }
        }

        public static readonly DependencyProperty OneWayProperty =
            DependencyProperty.Register("OneWay", typeof(string), typeof(MainWindow), new FrameworkPropertyMetadata("", FrameworkPropertyMetadataOptions.AffectsArrange));
       
```

尝试再定义两个属性，绑定依赖属性

```csharp
       public string Property1
        {
            get => _property; 
            set
            {
                _property = value;
                OnPropertyChanged();
            }
        }

        public string Property2
        {
            get => _property2; 
            set
            {
                _property2 = value;
                OnPropertyChanged();
            }
        }

        private string _property;
        private string _property2;

        public event PropertyChangedEventHandler PropertyChanged;

        private void OnPropertyChanged([CallerMemberName]string name = "")
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
        }
```

在构造函数绑定属性

```csharp
       public MainWindow()
        {
            DataContext = this;
            InitializeComponent();

            Binding binding = new Binding
            {
                Path = new PropertyPath("Property1"),
                Mode = BindingMode.Default
            };

            BindingOperations.SetBinding(this, TwoWayProperty, binding);

            binding = new Binding
            {
                Path = new PropertyPath("Property2"),
                Mode = BindingMode.Default
            };

            BindingOperations.SetBinding(this, OneWayProperty, binding);
        }
```

在界面绑定一下属性就知道属性是否修改

```csharp
        <StackPanel VerticalAlignment="Center" HorizontalAlignment="Center">
            <TextBlock Margin="10,10,10,10" Text="{Binding Property1}"></TextBlock>
            <TextBlock Text="{Binding Property2}" Margin="10,10,10,10"></TextBlock>
            <TextBlock x:Name="Text" Margin="10,10,10,10"></TextBlock>
            <Button Content="修改值" Click="Button_Click"></Button>
        </StackPanel>
```

界面的按钮点击的时候修改属性

```csharp
        private void Button_Click(object sender, RoutedEventArgs e)
        {
            Random ran = new Random();
            Text.Text = ran.Next().ToString();
            OneWay = Text.Text;
            TwoWay = Text.Text;
        }
```

运行代码点击按钮，可以发现只有 Property1 会修改

所有代码在 [github](https://github.com/lindexi/lindexi_gd/tree/d8adec7d80ebcc5bd019c2695b9788793286b2c7/CelakercalbochallhiNerjufeeqalchelfu)

建议只有在熟悉的属性才可以不写 Mode 防止翻车

[Explain Binding Mode In WPF](https://www.c-sharpcorner.com/article/explain-binding-mode-in-wpf/ )

[BindingMode Enum (System.Windows.Data)](https://docs.microsoft.com/en-us/dotnet/api/system.windows.data.bindingmode?wt.mc_id=MVP )

[.net - What are the defaults for Binding.Mode=Default for WPF controls? - Stack Overflow](https://stackoverflow.com/questions/1797105/what-are-the-defaults-for-binding-mode-default-for-wpf-controls )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
