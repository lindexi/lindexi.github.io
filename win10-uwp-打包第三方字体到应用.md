# win10 uwp 打包第三方字体到应用

有时候我们会把一些特殊字体打包到软件，因为如果找不到我们的字体会变为默认，现在很多字体图标我们用得好，有时候我们的应用会用很漂亮的字体，需要我们自己打包，因为用户一般是没有字体。

本文告诉大家如何在 UWP 内置字体，把字体打包到应用

<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>

## UWP使用第三方字体

首先我们需要字体，这个字体下载，网上很多。[http://font.chinaz.com/](http://font.chinaz.com/)我在这网站下载，如果有人知道他的网站字体不是完全授权，请和我说，我就删除我的应用字体。

在网上下东西很要注意版权，有很多是我们不能直接拿来用。

我们简单在界面写一个 TextBlock

```xml

        <TextBlock Margin="10,100,10,10"
                       Text="Lov ms"></TextBlock>

```

然后我们能运行看到一个字“Lov ms"因为我对微软的love没有了最后

![这里写图片描述](http://img.blog.csdn.net/20160918094918630)

为什么在Margin写`10,100,10,10`因为上面有黑色会挡住。

![这里写图片描述](http://img.blog.csdn.net/20160918094933677) 

第一步，我们去掉黑色左上角的调试，删除App.xaml.cs 下面的代码

```csharp
# if DEBUG
            if (System.Diagnostics.Debugger.IsAttached)
            {
                this.DebugSettings.EnableFrameRateCounter = true;
            }
# endif

```
第二步，
大家可以看到我们的应用没有特殊字体，这时我们写一个我们系统不存在的字体`And Love St.ttf` ，如何设置字体，请看代码

```xml

        <TextBlock Margin="10,100,10,10"
                   FontFamily="And Love St.ttf"
                   Text="Lov ms"></TextBlock>

```

于是运行程序可以看到如下图

![这里写图片描述](http://img.blog.csdn.net/20160918094918630)

几乎看不到有变化

我们添加一个文件夹，用于存放字体

![这里写图片描述](http://img.blog.csdn.net/20160918095356790) 

我把它叫 Font 

把我们的字体放入。建议直接拖进去，uwp 导入字体是把字体放到解决方案的文件夹

![这里写图片描述](http://img.blog.csdn.net/20160918095623994) 

在我们的字体类型写我们放进去字体，代码就是 `路径#字体名称`

假如我们的路径是`Font/And Love St.ttf` 名称`And Love st`，那么需要写的代码如下

注意路径是相对路径，和 页面放的位置有关。

```xml

        <TextBlock Margin="10,100,10,10"
                   FontFamily="Font/And Love St.ttf#And Love st"
                   Text="Lov ms" ></TextBlock> 

```

写完之后，运行一下程序，可以看到好看的字体

![这里写图片描述](http://img.blog.csdn.net/20160918100115278) 

参见：http://www.cnblogs.com/mycing/p/5658355.html

除了上面的方法，因为需要设置路径，比较难用，于是我提供一个方法，让大家可以简单使用字体。这个方法是把字体作为资源，这样比较容易写，请看代码

```xml

    <Page.Resources>
        <FontFamily x:Key="Font" >Font/And Love St.ttf#And Love st</FontFamily>
    </Page.Resources>

```

这样就可以在很多地方都使用，使用方法请看代码

```xml

        <TextBlock Margin="10,100,10,10"
                   FontFamily="{StaticReources Font}"
                   Text="Lov ms" ></TextBlock>

```

如果在后台代码需要使用设置字体，那么请使用下面的代码

```csharp
 textBlock.FontFamily = newFontFamily("/Assets/Swiftel.ttf#Swiftel Base DEMO");
```

路径和页面的方法一样，注意使用的是相对路径，和代码所在有关。如果已经写在资源，那么请看我的博客[win10 uwp 后台获取资源](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E5%90%8E%E5%8F%B0%E8%8E%B7%E5%8F%96%E8%B5%84%E6%BA%90/)

需要注意，UWP 不支持 OTF 字体。

参见：[UWP开发百科之---内置字体 - 快乐 就在你的心 的博客](https://kljzndx.github.io/My-Blog/2017/06/25/UWP%E5%BC%80%E5%8F%91%E7%99%BE%E7%A7%91%E4%B9%8B-%E5%86%85%E7%BD%AE%E5%AD%97%E4%BD%93/)

http://www.c-sharpcorner.com/article/custom-fonts-in-windows-10-uwp-app/

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。