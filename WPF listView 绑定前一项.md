# win10 uwp listView 绑定前一项

大神问，如何在 ListView 绑定前一项，于是我下面告诉大家如何在 ListView 绑定前一项

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->

<!-- csdn -->

## WPF 绑定前一项

可以使用绑定的 RelativeSource 就可以绑定前一项，请看代码

```csharp
        <ListView >
            <ListViewItem>
                <ListViewItem.Style>
                    <Style>
                        <Style.Triggers>
                            <DataTrigger Binding="{Binding RelativeSource={RelativeSource PreviousData}}">
                                
                            </DataTrigger>
                        </Style.Triggers>
                    </Style>
                </ListViewItem.Style>
            </ListViewItem>
        </ListView>
```

## UWP 绑定前一项

如果需要在ListView 让每个项绑定前一个项的内容，那么就是本文要说的。

首先有一个数据的类，我新建一个 Foo

```csharp
    public class Foo : INotifyPropertyChanged
    {
        private string _name;

        public string Name
        {
            get { return _name; }
            set
            {
                _name = value;
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

然后在界面做一个简单的列表，需要有两个TextBlock 一个绑定这一项的数据，一个绑定上一个项的数据

```csharp
        <ListView ItemsSource="{Binding Foo}">
            <ListView.ItemTemplate>
                <DataTemplate DataType="local:Foo">
                   <Grid>
                      <TextBlock Text="{Binding Name}" ></TextBlock>
                       <TextBlock x:Name="上一个的" Margin="10,100,10,10" Text="" ></TextBlock>
                   </Grid>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
```

后台代码需要创建一个数据，但是这个数据我不会在运行添加

```csharp
        public ObservableCollection<Foo> Foo { get; set; } = new ObservableCollection<Foo>()
        {
            new Foo()
            {
                Name = "lindexi"
            },
            new Foo
            {
                Name = "csdn"
            }
        };
```

这样看起来就是简单的代码，但是如果需要绑定上一项就需要添加一个新类

假如从后台拿到一个 TextBlock ，那么如何从这个 TextBlock 拿到这个的 DataContext ，可以获得他上一级的，虽然从这里拿到也可以

```csharp
          var grid = (FrameworkElement) text.Parent;

            var foo = (Foo) grid.DataContext;
```

那么如何从 Grid 拿到ListView ，如果拿到这个就可以拿到绑定的数据，所以就可以从绑定的数据拿到当前的上一项，然后绑定。

如果需要从 Grid 拿到 ListView ，简单的代码是一个循环

```csharp
   var temp = grid;
            while (!(temp is ListView))
            {
                temp = (FrameworkElement) VisualTreeHelper.GetParent(temp);
            }
```

这样就拿到了，那么拿到数据就可以绑定

```csharp
           var foo2 = (IEnumerable<Foo>) ((ListView) temp).ItemsSource;

            var n = foo2.ToList();

            if (n.IndexOf(foo) > 0)
            {
                Binding bind = new Binding("Name")
                {
                    Source = n[n.IndexOf(foo) - 1],
                };
                BindingOperations.SetBinding(text, TextBlock.TextProperty, bind);
            };
```

一开始如何拿到 TextBlock ，可以使用一个附加属性来拿

```csharp
       public static readonly DependencyProperty FooProperty = DependencyProperty.RegisterAttached(
            "Foo", typeof(object), typeof(Foo1), new PropertyMetadata(default(object), FooPropertyChangedCallback));

        public static void SetFoo(DependencyObject element, object value)
        {
            element.SetValue(FooProperty, value);
        }

        public static object GetFoo(DependencyObject element)
        {
            return (object) element.GetValue(FooProperty);
        }

                      <Grid>
                      <TextBlock Text="{Binding Name}" ></TextBlock>
                       <TextBlock x:Name="上一个的" Margin="10,100,10,10" Text="" local:Foo1.Foo="{Binding RelativeSource={RelativeSource Self}}"></TextBlock>
                   </Grid>
```

然后把代码写到 FooPropertyChangedCallback 就可以了

代码很简单，我就不写所有代码

代码：[[免费]ListViewBindLastItem 1.0-CSDN下载](http://download.csdn.net/download/lindexi_gd/9979367)