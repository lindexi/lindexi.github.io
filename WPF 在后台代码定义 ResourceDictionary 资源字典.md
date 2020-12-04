# WPF 在后台代码定义 ResourceDictionary 资源字典

在 WPF 中的 ResourceDictionary 资源字典大部分都是在 XAML 里面定义的，但是在 C# 代码定义一个资源字典也是可行的，只是写起来有点诡异

<!--more-->
<!-- CreateTime:2020/11/20 17:14:59 -->



在 CSharp 后台代码里面给 WPF 定义资源字典需要重新创建一个类，让这个类继承 ResourceDictionary 如以下代码

```csharp
    public class Foo : ResourceDictionary
    {

    }
```

然后在构造函数里面加入测试的代码，添加一个颜色作为资源

```csharp
    public class Foo : ResourceDictionary
    {
        public Foo()
        {
            Add("Foo", Brushes.Gray);
        }
    }
```

在后台代码添加的资源需要在被加入到使用之前，完成资源的添加，因此建议写在构造函数里面

使用这个在后台代码定义的资源字典有两个方法，一个是在 XAML 引用，另一个是在后台代码添加

在 XAML 引用的逻辑如下

```xml
    <Window.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <local:Foo></local:Foo>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Window.Resources>

    <Grid Background="{StaticResource Foo}">

    </Grid>
```

需要注意的是在 C# 后台定义的 WPF 资源字典不能通过 Url 的方式引用，而是需要通过实例的方式。可以选择创建实例或引用资源的方式，如上面代码是创建实例

在上面代码定义了一个测试使用的代码，尝试使用 `StaticResource Foo` 静态资源，这个静态资源是定义在后台代码的资源字典的，运行代码可以看到能绑定上

而在后台代码添加引用的方法如下

```csharp
    Resources.MergedDictionaries.Add(new Foo());
```

这就是在后台定义资源字典的用法了

在后台代码定义资源字典还有一个有趣的黑科技是重新返回资源的值

大概代码逻辑如下

```csharp
    public class Foo : ResourceDictionary
    {
        public Foo()
        {
            Add("Foo", Brushes.Gray);
        }

        protected override void OnGettingValue(object key, ref object value, out bool canCache)
        {
            value = Brushes.Blue;
            canCache = true;
        }
    }
```

在这个资源字典里面定义了 Foo 是灰色，但是在 OnGettingValue 方法里面返回的是蓝色。因此在 XAML 里面绑定静态资源的时候，将会显示的实际颜色是蓝色

另外只要资源里面的值不是 null 空，那么都会进入 OnGettingValue 方法去读取实际返回的值，而实际返回值是 object 类型，意味着可以愉快更改返回类型

```csharp
        public Foo()
        {
            Add("Foo", null); // 传入 null 将不会进入 OnGettingValue 方法
        }

        protected override void OnGettingValue(object key, ref object value, out bool canCache)
        {
            value = Brushes.Blue;
            canCache = true;
        }
```

如以下代码，传入的是 object 但是在 OnGettingValue 方法可以返回颜色

```csharp
        public Foo()
        {
            Add("Foo", new object());
        }

        protected override void OnGettingValue(object key, ref object value, out bool canCache)
        {
            value = Brushes.Blue;
            canCache = true;
        }
```

利用这个有趣的科技也可以用来做多语言或者主题色等，只是这个方法没有自动的更新值机制

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/9b4f948b/LojafeajahaykaWiweyarcerhelralya)欢迎小伙伴访问

而在 XAML 定义内容，同时支持对应的后台代码也可以，但是没有什么意义，至少我还不知道这个功能有什么作用

做法就是和上面代码一样，定义一个继承 ResourceDictionary 的类，如下面代码

```csharp
    public class ResourceJainahijainenelHuceenukur : ResourceDictionary
    {
        public ResourceJainahijainenelHuceenukur()
        {
            Debugger.Break();
        }

        protected override void OnGettingValue(object key, ref object value, out bool canCache)
        {
            Debugger.Break();
            base.OnGettingValue(key, ref value, out canCache);
        }
    }
```

接着创建一个资源字典，创建之后修改 ResourceDictionary 为刚才创建的类名

```xml
 <local:ResourceJainahijainenelHuceenukur xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                           xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                           xmlns:local="clr-namespace:KemkicemdurFemceloja">
    <SolidColorBrush x:Key="Foo" Color="Aquamarine"></SolidColorBrush>
</local:ResourceJainahijainenelHuceenukur>
```

此时就完成了，依然使用的时候使用 url 的形式

但是这样做我想不到有多少意义，因为 OnGettingValue 方法尽管重写了，但是实际不会被调用进入

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/d98030cf4a7c21b945466d993a4bfaf5f7cc477e/KemkicemdurFemceloja)欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
