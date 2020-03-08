# Xamarin.Forms 按钮样式 圆角按钮

在 Xamarin 中可以通过 CornerRadius 设置按钮使用圆角

<!--more-->
<!-- CreateTime:2020/3/1 10:35:13 -->

<!-- 发布 -->

在 Xamarin 中可以方便进行样式定义或不进行定义样式只修改属性而改变外观，如按钮的圆角可以通过 CornerRadius 属性设置

按钮使用圆角时，如果更改边框的颜色建议同时更改边框的宽度和边框颜色，在不同的平台下默认的样式不相同的，如果想要保持各个平台统一的外观，虽然这样不是好主意，那么请设置固定的值，而不是采用默认值

```xml
            <Button Font="Large" Text="选取PPT文件" HorizontalOptions="Center"
                    CornerRadius="5"
                    BackgroundColor="Transparent"
                    BorderColor="Aquamarine" 
                    BorderWidth="2"/>
```

此时就创建了一个圆角的按钮，注意需要设置边框时同时设置 BorderColor 和 BorderWidth 两个值。因为在 UWP 中 BorderWidth 是 2 而在 Android 中是 0 也就是此时如果干掉了背景颜色，将看不到按钮的圆角

<!-- ![](image/Xamarin.Forms 按钮样式 圆角按钮/Xamarin.Forms 按钮样式 圆角按钮0.png) -->

![](http://image.acmx.xyz/lindexi%2F202022318525421.jpg)

设置按钮背景透明可以通过设置 BackgroundColor 为 Transparent 属性

如果需要让按钮点击时呈现有趣的效果，可以通过 VisualStateManager 的方式定义

```xml
            <Button Font="Large" Text="选取PPT文件" HorizontalOptions="Center"
                    CornerRadius="5"
                    BackgroundColor="Transparent"
                    BorderColor="Aquamarine" 
                    BorderWidth="2">
                <VisualStateManager.VisualStateGroups>
                    <VisualStateGroup x:Name="CommonStates">
                        <VisualState x:Name="Normal">
                            <VisualState.Setters>
                                <Setter Property="Scale"
                                        Value="1" />
                            </VisualState.Setters>
                        </VisualState>

                        <VisualState x:Name="Pressed">
                            <VisualState.Setters>
                                <Setter Property="Scale"
                                        Value="0.6" />
                            </VisualState.Setters>
                        </VisualState>

                        <VisualState x:Name="Released">
                            <VisualState.Setters>
                                <Setter Property="Scale"
                                        Value="2" />
                            </VisualState.Setters>
                        </VisualState>
                    </VisualStateGroup>
                </VisualStateManager.VisualStateGroups>
            </Button>
```

这个项目所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/9fdafab123302ae7d7a2c9aecb590109218c4e72/JardalllojoHayeajemjuli) 欢迎小伙伴访问

另外推荐小伙伴的系列博客：[Xamarin移动开发之路 - peterYong - 博客园](https://www.cnblogs.com/peterYong/p/11589553.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
