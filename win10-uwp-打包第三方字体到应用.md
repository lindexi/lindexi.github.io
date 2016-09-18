#win10 uwp 打包第三方字体到应用

有时候我们会把一些特殊字体打包到软件，因为如果找不到我们的字体会变为默认，现在很多字体图标我们用得好，有时候我们的应用会用很漂亮的字体，需要我们自己打包，因为用户一般是没有字体。

首先我们需要字体，这个字体下载，网上很多。[http://font.chinaz.com/](http://font.chinaz.com/)我在这网站下载，如果有人知道他的网站字体不是完全授权，请和我说，我就删除我的应用字体。

在网上下东西很要注意版权，有很多是我们不能直接拿来用。

我们简单在界面写一个TextBlock

```

        <TextBlock Margin="10,100,10,10"
                       Text="Lov ms"></TextBlock>

```

然后我们能运行看到一个字“Lov ms"因为我对微软的love没有了最后

![这里写图片描述](http://img.blog.csdn.net/20160918094918630)

为什么在Margin写`10,100,10,10`因为上面有黑色会挡住。

![这里写图片描述](http://img.blog.csdn.net/20160918094933677) 

我们去掉黑色，删除App.xaml.cs

```
#if DEBUG
            if (System.Diagnostics.Debugger.IsAttached)
            {
                this.DebugSettings.EnableFrameRateCounter = true;
            }
#endif

```

大家可以看到我们的应用没有特殊字体，这时我们写一个我们系统不存在的字体`And Love St.ttf`

```

        <TextBlock Margin="10,100,10,10"
                   FontFamily="And Love St.ttf"
                   Text="Lov ms"></TextBlock>

```


![这里写图片描述](http://img.blog.csdn.net/20160918094918630)

几乎看不到有变化

我们添加一个文件夹

![这里写图片描述](http://img.blog.csdn.net/20160918095356790) 

我把它叫Font

把我们的字体放入，直接拖进去

![这里写图片描述](http://img.blog.csdn.net/20160918095623994) 

在我们的字体写我们放进去字体，路径#字体名称

我们的路径`Font/And Love St.ttf`名称`And Love st`

```

        <TextBlock Margin="10,100,10,10"
                   FontFamily="Font/And Love St.ttf#And Love st"
                   Text="Lov ms" ></TextBlock> 

```

![这里写图片描述](http://img.blog.csdn.net/20160918100115278) 

参见：http://www.cnblogs.com/mycing/p/5658355.html

 <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
