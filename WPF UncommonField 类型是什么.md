# WPF UncommonField 类型是什么

本文告诉大家一个黑科技，这个黑科技在.net 框架外无法使用，这就是 UncommonField 。下面将会告诉大家这个类有什么用。

<!--more-->
<!-- csdn -->

<!-- 标签：WPF，.net framework,.net源代码,源代码分析 -->

如果大家有反编译 UIElement 那么就会看到下面的代码

```csharp
internal static readonly UncommonField<EventHandlersStore> EventHandlersStoreField = new UncommonField<EventHandlersStore>();
```

那么这个`UncommonField`是什么？这个类是解决`DependencyObject `使用很多内存。使用这个类可以作为轻量的`DependencyObject `因为他使用很少的内存。

因为使用了`DependencyObject `就会创建很多默认的值，无论使用的是`TextBox.Text`的依赖属性还是`Grid.Row`附加的，都会有很多不需要使用的值。但是在框架，需要使用很少的内存，所以就使用`UncommonField`。

如果使用`UncommonField`就会去掉很多元数据、校验、通知，`UncommonField`会使用和`DependencyObject `相同的机制，让他可以存放在`DependencyObject `中和其他存放的属性一样，在没有改变值的时候会使用上一级、默认的值。所以可以减少一些内存。

因为现在很少人会写出和框架一样的那么多使用依赖属性，所以就不需要使用这个功能。

参见：https://stackoverflow.com/a/18280136/6116637

https://referencesource.microsoft.com/#WindowsBase/Base/System/Windows/UncommonField.cs



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
