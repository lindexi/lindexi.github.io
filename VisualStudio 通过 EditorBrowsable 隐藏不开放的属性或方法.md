# VisualStudio 通过 EditorBrowsable 隐藏不开放的属性或方法

在开发中，总是会有一些方法不期望让大家直接使用到，就可以通过 EditorBrowsable 特性让智能提示不显示这个属性或方法

<!--more-->
<!-- CreateTime:2018/12/15 10:30:35 -->

<!-- csdn  -->

假设我开发了这样一个类

```csharp
    public class Foo
    {
        public void Doubi()
        {
            Console.WriteLine("林德熙是逗比");
        }
    }
```

我不想让小伙伴调用 Doubi 方法，但是我自己又想使用，此时就可以使用 EditorBrowsable 标记在方法

```csharp
    public class Foo
    {
        [EditorBrowsable(EditorBrowsableState.Never)]
        public void Doubi()
        {
            Console.WriteLine("林德熙是逗比");
        }
    }
```

现在 VisualStudio 智能提示就不能够提示这个方法了，但是 Resharper 依然可以提示，只有通过 ReSharper > Options > Environment > IntelliSense > Completion Appearance 设置去掉 EditorBrowsable 的值才能不显示

于是现在小伙伴就无法从智能提示找到 Doubi 方法了，那么这个特性是在什么时候有用？在于自己写了一些不想让小伙伴用的属性或方法的时候

在 WPF 底层就在 DispatcherObject 的 CheckAccess 判断调用线程是否是创建线程的方法标记了这个特性，只有了解 WPF 依赖属性的小伙伴才能使用这个方法

当然这个做法没有接口隐藏的方法做的好，只是使用起来方便

[Resharper 配置](https://www.jetbrains.com/help/resharper/Reference__Options__Environment__IntelliSense__Completion_Appearance.html )

[EditorBrowsableAttribute Class](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.editorbrowsableattribute?view=netframework-4.7.2 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
