# WPF 拖动滚动

有时候需要这个效果，触摸向下可以向下滑动，不需要鼠标滑轮。

![](http://image.acmx.xyz/AwCCAwMAItoFAMV+BQA28wYAAQAEAK4+AQBmQwIAaOgJAOjZ/panel2017311092024.gif)

<!--more-->
<!-- CreateTime:2018/10/11 14:10:41 -->


<div id="toc"></div>

使用 ListBox 可以简单做到，那么如何使用 ScrollViewer 做到相同效果？

复制一下代码，运行可以看到，我的可以做到拖动时，自动滑动。


```csharp
          <ScrollViewer VerticalScrollBarVisibility="Visible"
                        PanningMode="VerticalOnly">
          <StackPanel>
                <TextBlock Text="林德熙"></TextBlock>
                <TextBlock Text="lindexi.oschina.io"></TextBlock>
                <TextBlock Text="blog.csdn.net/lindexi_gd"></TextBlock>
                <TextBlock Text="UWP"></TextBlock>
                <TextBlock Text="开发者"></TextBlock>
                <TextBlock Text="MS"></TextBlock>
                <TextBlock Text="csdn"></TextBlock>
                <TextBlock Text="滚动"></TextBlock>
                <TextBlock Text="点击"></TextBlock>
                <TextBlock Text="7.0"></TextBlock>
                <TextBlock Text="第一个是Out"></TextBlock>
                <TextBlock Text="Tuples"></TextBlock>
                <TextBlock Text="内部函数"></TextBlock>
            </StackPanel>
        </ScrollViewer>
```


其实就是设置`PanningMode="VerticalOnly"`

参见 [WPF 可触摸移动的ScrollViewer控件 - 唐宋元明清2188 - 博客园](https://www.cnblogs.com/kybs0/p/9766324.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 