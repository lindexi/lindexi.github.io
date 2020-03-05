# wpf 绑定 TextLength 

我看到朋友的代码出现绑定了 一个 TextBlock 的 Text 的 length ，那时候我觉得 length 不是依赖属性，绑定了是无法通知的。最后我做了实验才发现，原因有 Text 可以通知。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->

<!-- csdn -->

请看简单的代码，界面就是一个 TextBlock 和两个按钮，其中一个按钮是绑定了 length 如果大于 0 才可以使用。一个按钮是把 TextBlock 的文字设置为空或者设置为任意字符串。

```csharp
    <Grid>
        <Grid.Resources>
            <local:LengthToBoolenConverter x:Key="LengthToBoolenConverter"/>
        </Grid.Resources>
        <Grid.RowDefinitions>
            <RowDefinition Height="79*"/>
            <RowDefinition Height="28*"/>
        </Grid.RowDefinitions>
        <TextBlock x:Name="TitleBlock" Margin="0,0,-0.6,-0.2" ></TextBlock>
        <Grid Grid.Row="1">
            <Button Margin="10,10,10,10" HorizontalAlignment="Left" Content="确定" Click="ButtonBase_OnClick"></Button>
            <Button Margin="10,10,10,10" HorizontalAlignment="Right" IsEnabled="{Binding ElementName=TitleBlock,Path=Text.Length,Converter={StaticResource LengthToBoolenConverter}}" Content="确定"></Button>
        </Grid>
    </Grid>

      public class LengthToBoolenConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            var length = (int?) value;
            return length > 0;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }

            private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            if (string.IsNullOrEmpty(TitleBlock.Text))
            {
                TitleBlock.Text = "1";
            }
            else
            {
                TitleBlock.Text = "";
            }
        }
```

看到`IsEnabled="{Binding ElementName=TitleBlock,Path=Text.Length,Converter={StaticResource LengthToBoolenConverter}}"` 绑定就是 Text.Length ，但是 Length 不是依赖属性，没有通知，那么在 Text 变化时是否会通知？会的，因为使用的是 Text 的 Length，所以在 Text 修改时就会改变了 Length 。

如果有一个 Model 类，这个类是没有继承通知的，那么如何在里面的属性修改时，可以通知？一个方法就是在 ViewModel 使用 Model 属性，每次都是修改整个 Model

```csharp
    class Model
    {
        public string Foo { set; get}
        //其他就不写了
    }

    class ViewModel : INotifyPropertyChanged
    {
        public void Foo()
        {
            var model = new Model()
            {
                Foo = Model.Foo,
                //复制所有属性
            };
            model.Foo = "新的";
            Model = model;
        }

        private Model _model;

        public Model Model
        {
            get { return _model; }
            set
            {
                _model = value;
                OnPropertyChanged();
            }
        }

        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
```

这样可以不修改 Model 就可以在修改属性通知，但是可以看到这需要复制所有属性，不过我有写了[C# 使用Emit深克隆](http://lindexi.oschina.io/lindexi//post/C-%E4%BD%BF%E7%94%A8Emit%E6%B7%B1%E5%85%8B%E9%9A%86/)使用这个就可以做快复制，而且还是深复制，但是复制不是真的深，代码复制是一层，如果需要实现真的复制，还需要自己去写

那么不复制是否可以做？可以，只需要调用`OnPropertyChanged`

请把上面的代码做修改，添加一个函数，这个函数更新会自动通知。

```csharp
        public void Foo(string str)
        {
            Model.Foo = str;
            OnPropertyChanged(nameof(Model));
        }
```

在按钮点击就把代码修改为

```csharp
        private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            if (string.IsNullOrEmpty(TitleBlock.Text))
            {
                ViewModel.Foo("1");
            }
            else
            {
                ViewModel.Foo("");
            }
        }
```

需要添加一些绑定，请看代码

```csharp
        public ViewModel ViewModel { get; set; } = new ViewModel();

```

设置了 DataContext 之后就可以在界面绑定，这时可以看到和直接设置TextBlock的文字看起来是一样

```csharp
        <TextBlock x:Name="TitleBlock" Margin="0,0,-0.6,-0.2" Text="{Binding Model.Foo}"></TextBlock>

```

源代码：[[免费]WpfTextLengthdc-CSDN下载](http://download.csdn.net/download/lindexi_gd/9968341)