# win10 UWP 你写我读

想要电脑读出我们写的内容，在win10，很简单
其实这个技术在windows7就有了，但是现在 win10 写出一个你写我读的软件很简单。

我们需要一个类 `MediaElement` 来播放，因为 windows10 的M arkdown 软件用的不是很好，所有我自己写一个。
这个软件我用了你写我读，如果需要代码，请自己去下 https://github.com/lindexi/Markdown

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


![这里写图片描述](http://img.blog.csdn.net/20160229103133774)

点击![这里写图片描述](http://img.blog.csdn.net/20160229103154353) 读出文本

在使用`SpeechSynthesizer`需要代码功能点 麦克风 ，需要申请，申请方式我就不说啦

![这里写图片描述](http://img.blog.csdn.net/20160229103615577)

![这里写图片描述](http://img.blog.csdn.net/20160229103657266)

代码我放在 model ，可以看到代码很少就可以做出想要的功能

```
        private async void speech(string str, MediaElement media_element)
        {
            SpeechSynthesizer synthesizer = new SpeechSynthesizer();
            SpeechSynthesisStream stream = await synthesizer.SynthesizeTextToStreamAsync(str);
            media_element.SetSource(stream, stream.ContentType);
            media_element.Play();
        }
```

实例化`SpeechSynthesizer`，使用`SynthesizeTextToStreamAsync`把文本变为流，需要注意，这里使用时候是异步，所以需要等待。

可以使用`MediaElement`播放，`MediaElement`播放需要把流和格式放到`MediaElement`


```
media_element.Play();
```

```
 <MediaElement Grid.Row="0" x:Name="mediaelement" AutoPlay="True" Volume="1.0" />
```

如果想知道更多
Volume 声音，请看老周博客

参考：http://www.cnblogs.com/tcjiaan/

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。