# win10 uwp 圆角按钮

本文讲的是如何做圆角按钮，我们在UWP本来的按钮都是矩形，圆角Radius没有，所以本文就用简单方法去做圆角按钮。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

我们按钮需要圆角，而自带没有，其实做一个很简单，把原来的按钮变为背景透明，然后使用矩形圆角，这样就是圆角按钮。

![QQ图片20160708162533.png](https://ooo.0o0.ooo/2016/07/08/577f650178218.png)

按钮背景颜色透明，那么我们可以使用`Background="{x:Null}"`这时我们没有了背景，可以在按钮使用矩形圆角，然后写上我们需要的显示，当然上面图就是我们做的

```xml
                    <Button.Content>
                        <Grid>
                            <Viewbox>
                                <Grid>
                                    <Rectangle Fill="Gray" RadiusX="20" RadiusY="20">

                                    </Rectangle>
                                    <StackPanel Orientation="Horizontal">
                                        <!--<TextBlock Text="同步"/>-->
                                        
                                    </StackPanel>
                                </Grid>
                            </Viewbox>
                        </Grid>
                    </Button.Content>
```

这是按钮一般情况，还有鼠标移动在按钮上、按钮被点击这些需要改。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

