# dotnet 读 WPF 源代码笔记 为什么自定义的 UserControl 用户控件不能跨程序集继承

从设计上，用户控件 UserControl 就不是一个合适用来多次继承的类型，更不要说进行跨程序集继承自定义的 UserControl 用户控件。对于大部分的用户控件来说，都是采用组合现有的控件来实现的功能，本身应该被当成一个模块来进行使用。在 WPF 框架里面，从框架层阻止了开发者对自定义的 UserControl 用户控件跨程序集继承的逻辑，一旦尝试进行跨程序集继承，将在运行时抛出异常。本文将从源代码的角度告诉大家 WPF 框架是如何阻止跨程序集继承

<!--more-->

<!-- 博客 -->
<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->

先来写一些演示使用的代码，新建一个 WpfLibrary1 项目用来存放自定义的用户控件。在 WpfLibrary1 项目里面新建一个 UserControl1.xaml 的用户控件

接着再新建一个叫 RukarcaheenereRelchairnalfe 的 WPF 项目，在这里面写一个叫 Foo 类型，让 Foo 类型继承 UserControl1 用户控件

```csharp
public class Foo : UserControl1
{
    public Foo()
    {
    }
}
```

在 MainWindow.xaml 里，将 Foo 加入到界面

```xml
    <Grid>
        <local:Foo></local:Foo>
    </Grid>
```

运行代码，可以看到抛出 System.Windows.Markup.XamlParseException 异常，内容如下

```
Exception: 组件“RukarcaheenereRelchairnalfe.Foo”不具有由 URI“/WpfLibrary1;component/usercontrol1.xaml”识别的资源。
```

以上的异常的大概含义就是定义的 `/WpfLibrary1;component/usercontrol1.xaml` 所在的程序集和 Foo 所在的程序集不是相同的一个程序集，在 WPF 框架层面禁止跨程序集继承自定义用户控件。更本质来说是禁止跨程序集加载 XAML 定义的界面资源

本文测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/9bcae76c2910b4dfb4b1e0ba02d59876c614fbb1/RukarcaheenereRelchairnalfe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/9bcae76c2910b4dfb4b1e0ba02d59876c614fbb1/RukarcaheenereRelchairnalfe) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 9bcae76c2910b4dfb4b1e0ba02d59876c614fbb1
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 RukarcaheenereRelchairnalfe 文件夹

通过断点调试，可以看到这个异常是从 InitializeComponent 方法里面抛出的。而此 InitializeComponent 方法是 WPF 的生成代码，实际代码放在 xx.g.i.cs 文件里面，里面的代码大概如下

```csharp
        public void InitializeComponent() 
        {
            if (_contentLoaded) 
            {
                return;
            }
            _contentLoaded = true;
            System.Uri resourceLocater = new System.Uri("/WpfLibrary1;component/usercontrol1.xaml", System.UriKind.Relative);
            
            System.Windows.Application.LoadComponent(this, resourceLocater);
        }
```

实际会抛出异常的就是 System.Windows.Application.LoadComponent 这句代码

进入 WPF 的开源仓库，可以看到 LoadComponent 的实现如下，以下代码删掉了细节部分

```csharp
    public class Application : DispatcherObject, IHaveResources, IQueryAmbient
    {
        public static void LoadComponent(Object component, Uri resourceLocator)
        {
            Uri currentUri = new Uri(BaseUriHelper.PackAppBaseUri, resourceLocator);
            PackagePart part = GetResourceOrContentPart(resourceLocator);
            Stream stream = null;
            stream = part.GetSeekableStream();
            IStreamInfo bamlStream = stream as IStreamInfo;
            if (bamlStream == null || bamlStream.Assembly != component.GetType().Assembly)
            {
                throw new Exception(SR.Get(SRID.UriNotMatchWithRootType, component.GetType( ), resourceLocator));
            }

            // 忽略其他代码
        }
    }
```

传入的 `resourceLocator` 就是 `/WpfLibrary1;component/usercontrol1.xaml` 的值，拿到的 `bamlStream` 的程序集是 WpfLibrary1 程序集

而 component 是定义在 RukarcaheenereRelchairnalfe 项目的类型，自然拿到的 `component.GetType().Assembly` 就是 RukarcaheenereRelchairnalfe 程序集

于是在 WPF 框架里面判断的 `bamlStream.Assembly != component.GetType().Assembly` 成立，抛出异常

也就是说，在 UserControl1 里面，采用的 `/WpfLibrary1;component/usercontrol1.xaml` 是期望从 WpfLibrary1 程序集获取对应的 XAML 定义资源（准确来说是 BAML 资源）进行加载。但实际的调用类型，却发现是继承的类型，放在另一个程序集，不符合框架设计的预期，抛出异常

这就是为什么自定义的 UserControl 用户控件不能跨程序集继承的原因

在 WPF 的 LoadComponent 方法是比较复杂的，本文只是将里面相关代码写出来，具体是如何调用的，我是通过调试的方法了解的

调试的方式我录了视频放在哔哩哔哩，请看 [为什么自定义的 UserControl 用户控件不能跨程序集继承_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1D3411u7tf/)