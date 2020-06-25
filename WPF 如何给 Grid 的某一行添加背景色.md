# WPF 如何给 Grid 的某一行添加背景色

其实在 WPF 里面是不存在单独设置 Grid 的某一行的配色，但是想要达到这个视觉效果，可以通过 Border 配合做到

<!--more-->
<!-- 发布 -->

使用方法很简单，假设咱创建了一个简单的 WPF 的 Xaml 界面如下

```xml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="*"></RowDefinition>
            <RowDefinition Height="*"></RowDefinition>
            <RowDefinition Height="*"></RowDefinition>
        </Grid.RowDefinitions>
    </Grid>
```

此时需要设置这个 Grid 的某一行的背景颜色，可以通过在这一行放一个 Border 同时设置这个元素的背景色做到

在 Grid 的某一行放某个元素的做法就是放下一个元素，指定这个元素放在 Grid 的哪一行，请看下面代码

```xml
        <Border Grid.Row="1" Background="Gray"></Border>
```

此时上面的代码就指定放在了 Grid 的第一行，注意上面代码是从第0行开始计算的。上面代码就设置了 Grid 的第1行存在一个只有背景的 Border 元素，因此视觉效果就是 Grid 的第一行背景色是灰色

在 Grid 的某个行列里面可以存放无数个元素，元素层级在没有指定 Canvas.ZIndex 时将会按照编写的顺序设置

因此想要让 Border 作为某一行的 Grid 的背景色，就需要将这个 Border 在对比这一行的其他元素最先写。因此最先写的元素就放在现实的最后面，可以理解为有一个画笔在画布上画，先画的图形将会在画面的最下方

看到这里小伙伴是不是也就理解了如何在 Grid 里面的某一列添加背景色呢。其实给 Grid 的列添加背景色和给行添加背景色的方法是差不多的

通过 Border 加上背景色的方法不仅可以满足视觉效果，也是相对来说性能比较好的方法。另外 Border 的背景支持画刷，也就是不知是纯色，还支持图片等。这部分就需要小伙伴自己玩一下

这个项目所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/88d685fb9a09f1f1df7b40f080af01e9b6574ce7/WinemwhajallawLigawakuja) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
