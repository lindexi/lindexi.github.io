# win10 uwp 禁止编译器优化代码

有时候写了一些代码，但是在优化代码的时候出错，但是如果不优化代码，性能很差。如何让编译器不优化一段代码？

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->


<!-- csdn -->

一般发布的软件都会选优化代码，点击属性选择生成就可以看到优化代码

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201792713825.jpg)

假如有方法 Foo ，这个方法里面写了一些特殊代码，不想让编译器优化，那么可以如何做？

有一个特性，可以让编译器不优化这段函数，这个特性就是`MethodImpl`

```csharp
[MethodImpl(MethodImplOptions.NoOptimization | MethodImplOptions.NoInlining)]
private void MethodWhichShouldNotBeOptimized()
{
}
```

如果使用特性，必须是 .net 3.5 以上，这个特性是在 3.5 加的，之前没有

关于 MethodImplOptions 参见 https://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.methodimploptions(v=vs.110).aspx

参见：https://stackoverflow.com/a/38633044/6116637

最近看到这篇文章讲的很好 [深入了解 WPF Dispatcher 的工作原理（Invoke/InvokeAsync 部分） - walterlv](https://walterlv.github.io/post/dotnet/2017/09/26/dispatcher-invoke-async.html ) 

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201792713624.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。