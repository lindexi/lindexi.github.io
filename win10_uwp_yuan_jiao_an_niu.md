# win10 uwp 圆角按钮

本文讲的是如何做圆角按钮，我们在UWP本来的按钮都是矩形，圆角Radius没有，所以本文就用简单方法去做圆角按钮。

我们按钮需要圆角，而自带没有，其实做一个很简单，把原来的按钮变为背景透明，然后使用矩形圆角，这样就是圆角按钮。

![QQ图片20160708162533.png](https://ooo.0o0.ooo/2016/07/08/577f650178218.png)

按钮背景颜色透明，那么我们可以使用`Background="{x:Null}"`这时我们没有了背景，可以在按钮使用矩形圆角，然后写上我们需要的显示，当然上面图就是我们做的

```
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