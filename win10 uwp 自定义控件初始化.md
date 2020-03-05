# win10 uwp 自定义控件初始化

我遇到一个问题，我在 xaml 用了我的自定义控件，但是我给他设置了一个值，但是什么时候我才可以获得这个值？

本文告诉大家，从构造函数、loaded、Initialized 的调用过程。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->


<div id="toc"></div>

用最简单的方法创建一个自定义控件，然后在他里面写一个属性

```csharp
        public static readonly DependencyProperty AmameProperty = DependencyProperty.Register(
            "Amame", typeof(int), typeof(MargeGlx), new PropertyMetadata(default(int)));

        public int Amame
        {
            get { return (int) GetValue(AmameProperty); }
            set { SetValue(AmameProperty, value); }
        }
```

然后在 MainPage 的 xaml 使用它，给他一个属性值，这里给他 2

然后写 load 和 Initialized 函数。

```csharp
      private void OnLoaded(object sender, RoutedEventArgs e)
        {
            if (Amame == 2)
            {

            }
        }

        protected override void OnInitialized(EventArgs e)
        {
            base.OnInitialized(e);

            if (Amame == 2)
            {
                
            }
        }
```

需要知道的， loaded 是事件，可以在构造使用下面代码

```csharp
            Loaded += OnLoaded;

```

然后运行，查看他们的属性值是否修改

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20176169376.jpg)

可以看到在构造函数时，属性没有获得值。

而在 OnInitialized ，同样，可以看到属性也没有获得值。同时无法获得 ActualHeight 和 ActualWidth 的值。

在 Loaded 的时候，可以获得属性的值，而且可以获得 ActualHeight 和 ActualWidth 的值。

所以调用顺序是 构造-OnInitialized-Loaded



参见：[WPF概念解析一： FrameworkElement的Loaded事件和Initialized事件](http://www.cnblogs.com/tedzhao/archive/2011/11/08/WPF_FrameworkElement_LoadedAndInitialized.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
