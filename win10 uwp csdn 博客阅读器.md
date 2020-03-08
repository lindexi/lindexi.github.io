# win10 uwp csdn 博客阅读器

本文告诉大家如何写一个 rss 阅读器，并且用它来获得 csdn 博客。

因为 csdn 的首页有很多文章，但是我喜欢的文章很少，于是我需要一个可以过滤关键词的工具。如果文章包含了某些关键词就把文章给我，这样就可以减少我去找文章的时间。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<!-- csdn -->

为了做这个软件，我需要下面的技术：

 - [win10 uwp 读写XML - lindexi_gd的专栏 - CSDN博客](http://blog.csdn.net/lindexi_gd/article/details/71077198)

 - [win10 UWP RSS阅读器](https://lindexi.github.io/lindexi/post/win10-UWP-RSS%E9%98%85%E8%AF%BB%E5%99%A8/)

 - [win10 UWP ListView](https://lindexi.github.io/lindexi//post/win10-UWP-ListView/)

 - Piovt 的相关知识

一开始我使用的是 ms 的 Rss 类，但是很不好用，很多信息无法得到，于是我就对 csdn 使用了 xml 的方法解析。需要知道，因为我是对 csdn 博客使用 xml 解析，所以这个软件可能无法在其他地方使用，因为 csdn 使用的不是标准的。最少不是垃圾微软的标准，所以对于一些其他博客，可能解析就出错了。

项目放在 [全球交友平台](https://github.com/lindexi/UWP/tree/master/uwp/src/Boleslav) ，如果有兴趣 开发的话，请直接下载代码到本地，编辑之后提交。如何使用 github 参见 [Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

于是可以看到，我需要的软件只有两个页面，一个是显示博客，一个是设置。

为了简单，我也不上 MVVM 啦，直接的代码就是点击确定，扫描所有的 rss ，获取他们的内容，然后判断是否存在关键词，如果存在的话，就显示，如果不存在，就不显示。

软件的功能，获取csdn推荐的博客，包括首页和 web 、前端、编程语言这些页面的推荐博客，判断这些内容中的文章，是否包含了用户设置的关键字，如果包含了，那么就是判断这个文章是用户喜欢的，显示出来。

用户可以通过点击自己喜欢的文章，跳转到浏览器去看文章。

看起来软件的功能很简单，做起来基本没有遇到什么难的地方，但是界面我做的不好看，我觉得这个功能应该放在 之前的 csdn 阅读软件。不过因为今天已经到了吃饭的时间，我需要回去了，所以就没有做。这个软件是为了我可以在手机看 csdn 博客，因为是 UWP ，在手机上也可以使用。

本来想告诉大家，这个软件是如何做的，但是感觉这个会用很长时间，于是我就不多说啦，反正代码已经开源，大家可以去看看。

![](http://i.wotula.com/wp.png)



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 