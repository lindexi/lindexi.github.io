# win10 uwp 如何判断一个对象被移除

有时候需要知道某个元素是否已经被移除，在优化内存的时候，有时候无法判断一个元素是否在某个地方被引用，就需要判断对象设置空时是否被回收。

本文告诉大家一个简单的方法判断对象是否被移除。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->


在 C# 是不需要自己手工释放对象，微软会判断对象是否被引用，如果一个对象被引用了，那么就不会移除这个对象。

但是有一种引用是弱引用，虽然他引用了对象，但是垃圾回收是不会因为对象有弱引用就不移除他。所以可以使用弱引用判断对象是否被移除。

例如有一个对象 A ，这个对象在很多地方都使用，但是不确定在某个移除 A 的地方之后，是否A会被移除，于是可以使用下面的代码来判断A是否被移除。

```csharp
        private object Foo { set; get; } = new object();

        private WeakReference<object> FooReference { set; get; }

        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            FooReference = new WeakReference<object>(Foo);
            Foo = null;
            GC.Collect();

            var timer = new DispatcherTimer();
            timer.Interval = new TimeSpan(0, 0, 0, 5);
            timer.Tick += (o, args) =>
            {
                Console.WriteLine(FooReference.TryGetTarget(out var t));
            };
             timer.Start();
        }      
```

点击按钮可以看到，输出 false ，也就是 Foo 被移除了

上面的代码使用的就是添加一个`WeakReference`引用对象，然后在判断对象是否被移除时，尝试获得对象，如果不能获得，那么就是对象被移除。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171013131223.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。