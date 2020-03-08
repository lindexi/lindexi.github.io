# UWP WPF 解决 xaml 设计显示异常

本文告诉大家如何解决 xaml 设计显示异常

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->

虽然垃圾微软可以在写的时候直接让开发者看到界面，但是他的设计比较烂，总是无法使用

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F2018314111735.jpg)

或者

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F201831585054.jpg)

简单的方法是禁用项目代码，在左下角可以找到这个按钮 禁用代码

另一个方法是设置判断当前是否在设计，如果是就直接返回，一般写在构造函数

## WPF

例如我创建一个用户控件 TsjcyubtnTtqtjem 那么就可以在构造函数添加下面代码

```csharp
        public TsjcyubtnTtqtjem()
        {
            InitializeComponent();

            if (DesignerProperties.GetIsInDesignMode(this))
            {
                return;
            }

            foreach (var temp in typeof(Brushes)
                .GetProperties(BindingFlags.Static | BindingFlags.Public | BindingFlags.NonPublic)
                .Select(temp => temp.GetValue(null, null)))
            {
                var rectangle = new Rectangle
                {
                    Height = 20,
                    Fill = (Brush)temp
                };

                HcrkKmqnnfzo.Children.Add(rectangle);
            }
        }
```

上面代码实际使用 `DesignerProperties.GetIsInDesignMode` 判断当前是否设计，如果是就返回，然后关闭这个类再打开一般就可以继续写的时候获得界面。

## UWP

在 UWP 可以通过`Windows.ApplicationModel.DesignMode.DesignModeEnabled`判断，或者通过`DesignMode.DesignMode2Enabled`判断，因为 DesignMode2Enabled 需要秋季更新以上的才可以使用。

```csharp
        public AssBjPage()
        {
            this.InitializeComponent();
            if (Windows.ApplicationModel.DesignMode.DesignModeEnabled)
            {
                return;
            }
        }
```

上面的代码需要在所有的引用控件加上，一般的微软控件不需要，因为他里面有加上

![](https://i.loli.net/2018/07/01/5b3837806dddf.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
