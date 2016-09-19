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



