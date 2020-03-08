# win10 uwp 自定义控件入门

本文告诉大家如何在 UWP 使用 CustomControl 自定义控件，在 UWP 的自定义控件的中文翻译是模板化控件，通过自定义控件可以完全控制整个控件的布局和渲染。

<!--more-->
<!-- CreateTime:2018/10/22 9:47:54 -->

<!-- csdn -->

默认创建的自定义控件是没有带 xaml 的，如果想要让 CustomControl 可以使用 xaml 就需要引入主题的方法

下面就来告诉大家如何使用 xaml 来做界面

## 在 CustomControl 使用 xaml 写界面

在 UWP 主要的元素就是控件，可以说，整个 UWP 的界面都依靠控件画出来的。使用 xaml 可以快速画出好看的界面，而默认创建的 自定义控件和用户控件不一样，用户控件会带一个 xaml 直接修改就可以在设计器看到界面。

通过创建一个类继承 Control 类，我这里创建的是一个 Board 类

```csharp
public sealed class Board : Control
```

然后在相同的文件夹，创建一个资源字典 Board.xaml 这样可以对应资源字典和创建的控件

在资源字典先引用命名控件，我这里创建 Board 是在 lindexi.UWP.Framework 命名空间，就需要在资源字典引用`xmlns:local="using:lindexi.UWP.Framework"` 这样才可以拿到对应的控件

```csharp
namespace lindexi.UWP.Framework
{
    public sealed class Board : Control
    {

    }
}
```

添加一个 Style 指定为刚才创建的 Board 控件

```xml
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:lindexi.UWP.Framework">
    <Style TargetType="local:Board">

    </Style>
</ResourceDictionary>
```

在这里不添加 Key 就是默认所有的 Board 控件都使用这个样式，通过修改 Template 的方法添加控件

```xml
    <Style TargetType="local:Board">
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="local:Board">
                    <Grid HorizontalAlignment="Stretch"
                          VerticalAlignment="Stretch">
                        
                    </Grid>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
```

通过 ControlTemplate 的方法里面就和用户控件一样可以使用 xaml 写出界面，我这里就放一个 ContentControl 可以来定制

可以使用 ContentControl 的 Content 属性放入任意的 UIElement 都可以加入视觉树

```xml
   <Style TargetType="local:Board">
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="local:Board">
                    <Grid HorizontalAlignment="Stretch"
                          VerticalAlignment="Stretch">
                        <ContentControl x:Name="ContentControl" 
                                        HorizontalAlignment="Stretch"
                                        VerticalAlignment="Stretch"/>
                    </Grid>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
```

但是现在的代码还没完成，还需要在项目创建一个 Theme 文件夹，然后在这个文件夹里面添加 Generic.xaml 资源字典，从这个字典引用刚才创建的 Board 资源字典，才可以在使用的时候找到

在 Generic.xaml 资源字典只需要添加下面的代码

```xml
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:lindexi.github.io">

    <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="ms-appx:///lindexi/Framework/Board.xaml" />
    </ResourceDictionary.MergedDictionaries>
</ResourceDictionary>
```

需要注意 ResourceDictionary 的路径，修改为自己实际的控件的 xaml 文件的路径，注意这里必须使用 `ms-appx:///` 开头，文件使用的是相对于项目的路径，如果使用的是相对于这个文件的路径，就会在运行的时候，在某个类的构造函数告诉

```csharp
Failed to assign to property 'Windows.UI.Xaml.ResourceDictionary.Source' because the type 'Windows.Foundation.String' cannot be assigned to the type 'Windows.Foundation.Uri'.
```

虽然现在设置好了控件的 xaml 但是现在的 xaml 没有内容，需要在 Board 类添加一些代码，让大家可以看到自己的 xaml 是否可以在 Board 使用

首先是添加 TemplatePart 在 Board 类，这样是在约定在 xaml 界面需要添加一个对应的控件，指定了控件的 Name 和这是一个什么控件

```csharp
    [TemplatePart(Name = "ContentControl", Type = typeof(ContentControl))]
    public sealed class Board : Control
```

是否记得在 Board 的资源字典就写了一个 ContentControl 类，虽然添加了约定但是还是需要将这个控件拿出来，通过重写 OnApplyTemplate 方法就可以使用 GetTemplateChild 方法拿到 xaml 里写的控件

```csharp
        protected override void OnApplyTemplate()
        {
            Content = (ContentControl) GetTemplateChild("ContentControl");
        }
```

这样就可以拿到对应的 xaml 的控件，虽然界面都在不断变化，但是这里拿到的控件是需要使用强转的方式，一旦找不到控件就给一个异常。

如果在 xaml 忘记写了一个控件，通过 GetTemplateChild 方法会返回 null 而不是抛异常，但是建议在这个方法下面判断拿到的如果是空，就抛出异常

```csharp
        protected override void OnApplyTemplate()
        {
            var foo = GetTemplateChild("不存在的控件");
            if (foo == null)
            {
                throw new ArgumentException("使用的模板不包含");
            }
        }
```

我通过去拿一个不存在的控件，拿到的是空判断是空就抛出异常

如果此时运行了代码，在 OnApplyTemplate 添加断点，会发现这个函数无法进来，原因是 Board 控件的构造函数还忘记写下面的代码

```csharp
        public Board()
        {
            this.DefaultStyleKey = typeof(Board);
        }
```

通过这个方法就可以拿到在 xaml 定义的控件，拿到了之后就可以在代码修改，如何修改请看下面

## 布局

如果已经写了 xaml 在代码拿到了 xaml 的控件，自定义控件还可以修改布局的方式

先在界面添加一些元素

```csharp
        public ContentControl Content { get; set; }

        private Grid _grid;

        protected override void OnApplyTemplate()
        {
            Content = (ContentControl) GetTemplateChild("ContentControl");
            _grid = new Grid()
            {
                Children =
                {
                    new TextBlock()
                    {
                        Text = "欢迎访问我博客 lindexi.github.io 里面有很多 UWP WPF 博客",
                        HorizontalAlignment = HorizontalAlignment.Center,
                        VerticalAlignment = VerticalAlignment.Center
                    }
                }
            };
            Content.Content = _grid;

            base.OnApplyTemplate();
        }
```

通过重写 MeasureOverride 方法可以拿到测量的值，在 UWP 的布局和 WPF 的一样，都是先进过测量再进行控制每个控件的坐标和大小。

测量是什么？在 UWP 的布局过程，这里提高了布局过程，还需要继续解释一下什么是布局过程。在 UWP 会将所有的控件按照控件所在的容器，作为视觉树，视觉树的意思很简单，我有一个 Grid 在里面放在两个 Grid 同时又在第一个 Grid 里面添加一个文本，这时的控件可以使用树这个数据结构表示。第一个节点是最上面的，也是最外层的 Grid 这个 Grid 有两个子节点，分别就是放在 Grid 里面的两个 Grid 而这里的两个 Grid 的第一个 Grid 里面也有一个节点就是文本。

在 UWP 通过 xaml 界面就可以知道控件的树结构，如果熟悉树这个结构就知道，可以使用递归的方式处理。也就是一个节点只处理这个节点的子节点，而不处理子节点的子节点，所以 UWP 的布局就依赖这个视觉树，通过布局子节点的方式，然子节点自己递归这个布局方法，布局子节点的子节点。

那么布局是什么？布局就是让子节点控件放在该放的地方，虽然定义了视觉树，知道了一个控件的里面包含了哪些控件，但是这个控件还没准备好里面的控件的坐标和大小。例如我有一个容器是 StackPanel 这个容器需要让里面的控件按照垂直或水平的方式布局，也就是在 StackPanel 垂直布局里面的控件，第二个控件的坐标的 Y 点是第一个控件的坐标的 Y 点加上控件的高度。假如第一个控件也是一个容器，那么如何知道这个容器的的高度是多少？因为容器的大小可以是容器里面的元素决定的，需要让这个容器先知道他里面的控件的大小才可以知道容器的大小。

这就是测量的过程，测量的过程就是让每个控件知道子节点的大小，从而计算出控件的大小，然后将控件的大小返回给上一层，让上一层可以知道子节点的大小。有了测量的过程，在进行 StackPenel 布局的时候，就可以在测量的过程知道了控件大小，从而在可以安排每个控件坐标。

这里自定义的控件也是这样，通过重写 MeasureOverride 可以修改计算自定义控件的大小的方法，从而报告给上一层一个特殊的值。

如我这里的控件是想要上一层给我多大的空间，我就要多大的空间，我可以通过重写 MeasureOverride 方法，返回参数

```csharp
        protected override Size MeasureOverride(Size availableSize)
        {
            base.MeasureOverride(availableSize);
            return availableSize;
        }
```

因为我这个控件里面有一些控件是需要在测量的过程重新给他一个值，我就可以这样写

```csharp
        protected override Size MeasureOverride(Size availableSize)
        {
            _grid.Height = availableSize.Height;
            _grid.Width = availableSize.Width;

            base.MeasureOverride(availableSize);
            return availableSize;
        }
```

处理测量的方法可以重写，布局的方法也可以重写

通过重写 ArrangeOverride 的方法可以做到实际的布局，从测量的方法传入的参数也许不是最外层控件在布局的时候传入的大小，假如我有一个 StackPanel 他的高度 100 宽度也是 100 在测量的过程就会传入大小是 100x100 但是在布局的过程就依赖当前的控件是在 StackPanel 的第几个控件，减去前面控件用的地方在是这个控件可以用的。

本文的控件是不需要重新布局的方法，现在看起来的控件的代码请看下面

```csharp
    [TemplatePart(Name = "ContentControl", Type = typeof(ContentControl))]
    public sealed class Board : Control
    {
        public Board()
        {
            this.DefaultStyleKey = typeof(Board);
        }

        protected override Size MeasureOverride(Size availableSize)
        {
            _grid.Height = availableSize.Height;
            _grid.Width = availableSize.Width;

            base.MeasureOverride(availableSize);
            return availableSize;
        }

        public ContentControl Content { get; set; }

        private Grid _grid;

        protected override void OnApplyTemplate()
        {
            Content = (ContentControl) GetTemplateChild("ContentControl");
            _grid = new Grid()
            {
                Children =
                {
                    new TextBlock()
                    {
                        Text = "欢迎访问我博客 lindexi.github.io 里面有很多 UWP WPF 博客",
                        HorizontalAlignment = HorizontalAlignment.Center,
                        VerticalAlignment = VerticalAlignment.Center
                    }
                }
            };
            Content.Content = _grid;

            base.OnApplyTemplate();
        }
    }
```

在界面添加这个控件然后运行一下，可以看到界面居中显示了这个控件

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
