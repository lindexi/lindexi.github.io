# win10 uwp xaml 绑定接口

本文告诉大家如何在 xaml 绑定属性使用显式继承接口。

<!--more-->
<!-- CreateTime:2018/6/15 21:07:19 -->


早上[快乐 就在你的心](https://kljzndx.github.io/My-Blog/ )问了我一个问题，他使用的属性是显式继承，但是无法在xaml绑定

我写了简单的代码，一个接口和属性

```csharp
    public class Foo : INotifyPropertyChanged, IF1
    {
        public Foo(string name)
        {
            _name = name;
        }

        private string _name;
        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }

        string IF1.Name
        {
            get { return _name; }
            set { _name = value; OnPropertyChanged(); }
        }

    }

    public interface IF1
    {
        string Name { set; get; }
    }
```

然后我尝试写一个列表，在前台绑定

```csharp
      public ObservableCollection<Foo> Foo { set; get; } = new ObservableCollection<Foo>()
        {
            new Foo("jlong"){}
        };
```

```csharp
        <ListView ItemsSource="{x:Bind Foo}">
            <ListView.ItemTemplate>
                <DataTemplate x:DataType="local:Foo">
                    <TextBlock Text="{Binding Path=Name }"></TextBlock>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
```

但是这样写出现绑定错误，因为在 Foo 是找不到 Name 属性，需要使用 IF1.Name 去拿到

我修改了代码

```csharp
                    <TextBlock Text="{Binding (local:IF1.Name)}"></TextBlock>

```

但是运行就出现了异常，说未指定，最后我尝试了新的方法，居然就编译通过，下面让我来告诉大家如何使用这个方法

```csharp
                    <TextBlock Text="{x:Bind Path=(local:IF1.Name) }"></TextBlock>

```

如果使用显式继承，那么在使用的时候需要使用他的接口来拿，但是接口不是直接写，需要先写空间，一般空间是写在最上，请看下面代码

```csharp
<Page
    x:Class="JoleenOneal.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:JoleenOneal"  这是空间
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">
```

然后需要加上括号，才可以使用

为什么上面的代码无法使用，现在我还不知道。

我找到了下面的观点

> The data binding team discussed adding support for interfaces a while ago but ended up not implementing it because we could not come up with a good design for it. The problem was that interfaces don't have a hierarchy like object types do. Consider the scenario where your data source implements both `IMyInterface1` and `IMyInterface2` and you have DataTemplates for both of those interfaces in the resources: which DataTemplate do you think we should pick up?

> When doing implicit data templating for object types, we first try to find a `DataTemplate` for the exact type, then for its parent, grandparent and so on. There is very well defined order of types for us to apply. When we talked about adding support for interfaces, we considered using reflection to find out all interfaces and adding them to the end of the list of types. The problem we encountered was defining the order of the interfaces when the type implements multiple interfaces.

> The other thing we had to keep in mind is that reflection is not that cheap, and this would decrease our perf a little for this scenario.

> So what's the solution? You can't do this all in XAML, but you can do it easily with a little bit of code. The `ItemTemplateSelector` property of `ItemsControl` can be used to pick which `DataTemplate` you want to use for each item. In the `SelectTemplate` method for your template selector, you receive as a parameter the item you will template. Here, you can check for what interface it implements and return the `DataTemplate` that matches it.

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  