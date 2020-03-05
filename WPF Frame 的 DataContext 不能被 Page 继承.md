# WPF Frame 的 DataContext 不能被 Page 继承

本文告诉大家在 Frame 的 DataContext 不能被 Page 继承如何解决。

<!--more-->
<!-- CreateTime:2018/6/11 10:48:24 -->

<!-- csdn -->

如果大家有研究 Frame 会发现一个诡异的现象。

假设 page 是在 Frame 里面的 Page ，通过下面的代码是可以拿到 DataContext ，而且假设 Frame 的 DataContext 就是一个定义的类 Foo 

```csharp
var frame = page.Parent as Frame;
// frame.DataContext == foo
```

但是如何直接拿 page 的 DataContext ，返回空。

原因是 Frame 是做了 Frame 里面的元素的 UI 隔离，也就是 DataContext 不能继承。

解决的方法是在 Frame 的 LoadCompleted 添加让里面元素知道 DataContext ，需要后台代码

```csharp
<Frame Name="frame"
       LoadCompleted="Frame_LoadCompleted"
       DataContextChanged="Frame_DataContextChanged"/>
```

```csharp
private void Frame_DataContextChanged(object sender, DependencyPropertyChangedEventArgs e)
{
    UpdateFrameDataContext(sender, e);
}
private void Frame_LoadCompleted(object sender, NavigationEventArgs e)
{
    UpdateFrameDataContext(sender, e);
}
private void UpdateFrameDataContext(object sender, NavigationEventArgs e)
{
    var content = frame.Content as FrameworkElement;
    if (content == null)
    {
        return;
    }
    
    content.DataContext = frame.DataContext;
}
```

参见：[c# - page.DataContext not inherited from parent Frame? - Stack Overflow](https://stackoverflow.com/questions/3621424/page-datacontext-not-inherited-from-parent-frame )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
