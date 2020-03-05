# win10 uwp 让焦点在点击在页面空白处时回到textbox中

在[网上](http://ask.csdn.net/questions/673861 ) 有一个大神问我这样的问题：在做UWP的项目，怎么能让焦点在点击在页面空白处时回到textbox中？

虽然我的小伙伴认为他这是一个 xy 问题，但是我还是回答他这个问题。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


首先需要知道什么是空白处，例如有下面的代码

```
 <Grid><TextBox x:Name="XcjKfqnkor"></TextBox></Grid>
```

那么空白就是 Grid ，需要点击他的时候回到 TextBox ，下面的代码我没有跑，可能无法运行

需要让 Grid 可以获得点击，需要给他背景

```
 <Grid Background="#01FFFFFF"><TextBox x:Name="XcjKfqnkor"></TextBox></Grid>
```

然后给 Grid 一个名称 

```
 <Grid x:Name="VkyZqfs" Background="#01FFFFFF"><TextBox x:Name="XcjKfqnkor"></TextBox></Grid>
```
在后天代码添加按下空白地方让 XcjKfqnkor 获得焦点

```csharp
Grid.AddHandler(PointerPressedEvent,
                new PointerEventHandler(Grid_OnPointerPressed), true);

         private void Grid_OnPointerPressed(object sender, PointerRoutedEventArgs e)
        {
            XcjKfqnkor.Focus();
        }
```

参见：[win10 uwp 获取按钮鼠标左键按下 - 林德熙](https://lindexi.oschina.io/lindexi/post/win10-uwp-%E8%8E%B7%E5%8F%96%E6%8C%89%E9%92%AE%E9%BC%A0%E6%A0%87%E5%B7%A6%E9%94%AE%E6%8C%89%E4%B8%8B.html )

[UWP开发大坑之---路由事件 - 快乐 就在你的心 的博客](https://kljzndx.github.io/My-Blog/2017/05/04/UWP%E5%BC%80%E5%8F%91%E5%A4%A7%E5%9D%91%E4%B9%8B-%E8%B7%AF%E7%94%B1%E4%BA%8B%E4%BB%B6/ )

如果是技术问题，建议到 Stackoverflow 提问，在csdn提问暂时比较少看到大神在看

欢迎加入Q群 53078485 讨论 uwp

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 