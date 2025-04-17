
# dotnet 6 使用 DependentHandle 关联对象生命周期

本文将告诉大家在 dotnet 6 新加入的 System.Runtime.DependentHandle 的类型的使用方法，通过 DependentHandle 可以实现将某个对象的引用生命周期和另一个对象关联起来

<!--more-->


<!-- CreateTime:2022/6/28 8:28:35 -->

<!-- 发布 -->

如 DependentHandle 结构体的构造函数，要传入两个对象作为参数，这两个对象参数分别是 target 和 dependent 参数对象，表示的意义是将这两个对象通过 DependentHandle 结构体建立关联。让 target 对象关联上 dependent 对象的生命周期，在 dependent 对象没有被释放之前，不会先释放 target 对象。功能上和 ConditionalWeakTable 差不多，只是 DependentHandle 作为 ConditionalWeakTable 的底层实现支持，直接使用 DependentHandle 可以有更多的控制

原本咱可以使用 ConditionalWeakTable 将对象进行关联，实现到将某个对象关联到另一个对象的生命周期上，只要另一个对象没有被释放，那么关联的对象也不会被释放。如此可以在不改动原有代码的前提下，让两个毫不关联的对象进行关联。例如可以用来实现缓存模块的功能

然而 ConditionalWeakTable 算是一个上层的封装，如果想要做更多的定制功能，那就可以采用在 dotnet 6 新加入的 System.Runtime.DependentHandle 结构体。这个结构体提供的是一对一的关联关系，而且可以方便调用 Dispose 方法解除关联性

接下来将写一个例子来告诉大家 DependentHandle 类型的使用方法

新建一个 WPF 项目，在界面放三个按钮，分别是进入断点按钮，用于进入断点看对象是否被释放。释放引用按钮，将关联的对象释放引用。由于在 .NET 里面，释放引用不代表立刻被回收，再加上一个 GC 按钮，用来点击的时候触发 GC 逻辑

```xml
        <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center" Orientation="Horizontal">
            <StackPanel.Resources>
                <Style TargetType="Button">
                    <Setter Property="Margin" Value="10,10,10,10" />
                </Style>
            </StackPanel.Resources>
            <Button Click="Button_Click">
                进入断点
            </Button>
            <Button x:Name="FreeObjectButton" Click="FreeObjectButton_Click">
                释放引用
            </Button>
            <Button x:Name="GCButton" Click="GCButton_Click">
                GC
            </Button>
        </StackPanel>
```

在后台代码里面定义 Foo1 和 Foo2 两个类型，定义类型的用途是在于方便在内存里面找到这两个类型的对象

```csharp
public class Foo1
{

}

public class Foo2
{

}
```

这两个类型没有相互关联性，在 MainWindow 添加一个 Foo1 类型的字段，如此即可让 Foo1 类型的对象被 MainWindow 引用，不会被释放

```csharp
    public MainWindow()
    {
        InitializeComponent();

        Loaded += MainWindow_Loaded;
    }

    private void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        var foo1 = new Foo1();

        _foo1 = foo1;
    }

    private Foo1? _foo1;
```

如果在 MainWindow_Loaded 方法里面，创建一个 Foo2 对象，而让这个对象没有地方引用，那自然很快就会 Foo2 对象就会被释放

```csharp
    private void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        var foo1 = new Foo1();

        new Foo2();

        _foo1 = foo1;
    }
```

如果将 Foo2 通过 DependentHandle 实现和 Foo1 关联，加入到关联列表里面，如以下逻辑，将可以让 Foo2 不会被释放

```csharp
    private void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        var foo1 = new Foo1();

        HandleList.Add(new DependentHandle(foo1, new Foo2()));

        _foo1 = foo1;
    }

    private List<DependentHandle> HandleList { get; } = new List<DependentHandle>();
```

假如 DependentHandle 是另外的某些类型，对 Foo2 的对象也有强引用关系，那自然 Foo2 不会被释放，只是不会被释放的理由是被 HandleList 给引用。但本文告诉大家的 DependentHandle 是不会对 Foo2 有强引用的，只是对 Foo2 关联 Foo1 对象

实现释放引用按钮的功能，点击按钮，将 Foo1 的对象引用释放，如此即可让 Foo1 的对象可以被回收

```csharp
    private void FreeObjectButton_Click(object sender, RoutedEventArgs e)
    {
        _foo1 = null;
    }
```

由于在 .NET 里面，不会立刻回收，实现 GC 按钮的功能，手动触发回收

```csharp
    private void GCButton_Click(object sender, RoutedEventArgs e)
    {
        GC.Collect();
        GC.WaitForFullGCComplete();
    }
```

实现断点按钮的逻辑

```csharp
    private void Button_Click(object sender, RoutedEventArgs e)
    {
        Debugger.Break();
    }
```

运行代码，先点击一下释放按钮，让 `_foo` 释放引用，接着点击 GC 按钮触发回收逻辑。多次点击 GC 按钮，等待一下，然后点击进入断电按钮，自己打上断点，通过 VS 的内存调试，可以看到 Foo1 和 Foo2 对象都没有存在内存里面

同时在 HandleList 列表里面，可以看到有一个 DependentHandle 对象，但是此对象里面的 Target 和 Dependent 都是 null 空对象

这就是证明了，通过 DependentHandle 可以建立从 Target 到 Dependent 的引用关联，在 Dependent 被回收之前，不会回收 Target 对象。在 Dependent 被回收之后，才会回收 Target 对象

在经过测试，使用 DependentHandle 的回收速度是比较慢的，很多时候，不是第一次点击 GC 按钮进行回收就能回收掉 Foo1 和 Foo2 对象的，而是需要多次点击

使用 dotnet 6 加入的 DependentHandle 进行底层的控制，更好写出符合自己业务逻辑的对象关联逻辑。例如实现自己的缓存库等。这个 Dependent 的功能是需要 CLR 层面提供的，也就是说这个类型只能在 dotnet 6 和更高版本使用，详细请看 [dotnet ConditionalWeakTable 的底层原理](https://blog.lindexi.com/post/dotnet-ConditionalWeakTable-%E7%9A%84%E5%BA%95%E5%B1%82%E5%8E%9F%E7%90%86.html )

更多请看 [DependentHandle Struct (System.Runtime) Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.dependenthandle?view=net-6.0&WT.mc_id=WD-MVP-5003260 )

以上的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/8378751cc0766bc804fa8a1ae5ef3e0766dd5b99/LemjallbabelyeeburKemkubejer) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/8378751cc0766bc804fa8a1ae5ef3e0766dd5b99/LemjallbabelyeeburKemkubejer) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 8378751cc0766bc804fa8a1ae5ef3e0766dd5b99
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 8378751cc0766bc804fa8a1ae5ef3e0766dd5b99
```

获取代码之后，进入 LemjallbabelyeeburKemkubejer 文件夹





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。