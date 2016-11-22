#CSDN 阅读 源代码

【】

我想得到我CSDN博客的阅读量，那么我应该做一个软件，这个软件可以查看。

##ListView宽度过小

这个问题简单。

```
                <ListView.ItemContainerStyle>

                    <Style TargetType="ListViewItem">

                        <Setter Property="HorizontalContentAlignment"

                                Value="Stretch"></Setter>

                    </Style>

                </ListView.ItemContainerStyle>
```

![QQ截图20160909195739.png](https://ooo.0o0.ooo/2016/09/09/57d2a38d3dc4c.png)

我们可以使用我们的ListView放数据



##获取博客

获取博客可以访问网站，获取源码，使用匹配到的数据

我们写软件，一般是用用一个页面来做导航，这个页面就是一个Frame，然后包含各种导航，所以这个页面会一直存在我们的内存。

然后我们需要把MainPage一开始就导航到我们这个页面，我们可以拿到我们的Content，然后把Content给一个Frame，用Frame导航。

```

        public MainPage()
        {
            this.InitializeComponent();
            Frame frame=Content as Frame;
            if (frame == null)
            {
                frame=new Frame();
                Content = frame;
            }
            frame.Navigate(typeof(View.AssBjPage));
        }

```

`if (frame == null)`一定会true，因为Content 一般是Grid，我们把content改为Frame

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

