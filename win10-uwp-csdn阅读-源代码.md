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

