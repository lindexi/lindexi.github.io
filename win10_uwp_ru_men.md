# win10 uwp 入门

UWP是什么我在这里就不说，本文主要是介绍如何入门UWP，也是合并我写的博客。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

关于UWP介绍可以参见:http://lib.csdn.net/article/csharp/32451

首先需要申请一个微软账号，没有账号是没法上传软件。

申请可以看：http://blog.csdn.net/lindexi_gd/article/details/50329121

之后我们可以在官网下载vs，这个不需要多说，但是vs安装一般使用自定义，选择全平台，把所有可以打钩加上，当然安卓什么跨平台就不要。

忘了，其实我们还需要windows10 系统。

学习开发一般学Hellow，那么国内看到一篇比较好的 http://www.ceix.me/suibi/uwp开发学习笔记-1-hello-world

官方 https://msdn.microsoft.com/zh-cn/windows/uwp/get-started/create-a-hello-world-app-xaml-universal

开发开始就是我们的控件，一般控件可以参见：http://blog.csdn.net/lindexi_gd/article/details/50972343

当然还有：http://blog.csdn.net/NoMasp/article/details/50263383

接着我们就需要学如何使用控件，参见：http://blog.csdn.net/lindexi_gd/article/details/50964889

控件，我们有时觉得官方的不够，我们需要学自定义，可以看[win10 UWP button](http://blog.csdn.net/lindexi_gd/article/details/50450292) [ win10 UWP FlipView](http://blog.csdn.net/lindexi_gd/article/details/50272907) [RichEditBox 使用自定义菜单](http://blog.csdn.net/lindexi_gd/article/details/50250795)

我们需要知道一些新的UWP带来的，[x:bind](http://blog.csdn.net/lindexi_gd/article/details/48294123) 和加载 http://blog.csdn.net/lindexi_gd/article/details/49743845

win10 UWP 显示地图 http://blog.csdn.net/lindexi_gd/article/details/49935341

当然我们和用户之间还需要MessageDialog 和 ContentDialog，http://blog.csdn.net/lindexi_gd/article/details/50822507

在用到APPBarButton他的Icon我都写出： http://blog.csdn.net/lindexi_gd/article/details/49307913

如果觉得微软控件还是不够，可以来看下我做的一些控件，圆形等待 http://blog.csdn.net/lindexi_gd/article/details/50606261

我做的SplitViewItem http://blog.csdn.net/lindexi_gd/article/details/51784671

如果需要图床 http://blog.csdn.net/lindexi_gd/article/details/51784666

如果你也做了一些好用的，希望给别人可以看 http://www.win10.me/?p=952

win10 改了很多，最简单的读写文件：http://blog.csdn.net/lindexi_gd/article/details/49007841，还有比较少用的md5 http://blog.csdn.net/lindexi_gd/article/details/48951849 ,Hamc http://blog.csdn.net/lindexi_gd/article/details/50830924

我们有一些常用的，这些可以看下，虽然不是马上可以用到

- win10 UWP 全屏 http://blog.csdn.net/lindexi_gd/article/details/51093890
 
- win10 uwp 屏幕常亮 http://blog.csdn.net/lindexi_gd/article/details/51166285
 
- win10 uwp 判断文件存在 http://blog.csdn.net/lindexi_gd/article/details/51387901
 
- win10 UWP 标题栏后退 http://blog.csdn.net/lindexi_gd/article/details/50618029
 
- 剪贴板 http://blog.csdn.net/lindexi_gd/article/details/50479180
 
- win10 UWP 应用设置 http://blog.csdn.net/lindexi_gd/article/details/50506692
 
- win10 UWP 获取系统信息 http://blog.csdn.net/lindexi_gd/article/details/50277341
 
- win10 uwp clone http://blog.csdn.net/lindexi_gd/article/details/50117925
 
- UWP 绘制图形 http://blog.csdn.net/lindexi_gd/article/details/49805029
  
- win10 uwp 通知Toast http://blog.csdn.net/lindexi_gd/article/details/49824613
  
- UWP xaml 圆形头像 http://blog.csdn.net/lindexi_gd/article/details/49757187
 
- win10 uwp App-to-app communication 应用通信 http://blog.csdn.net/lindexi_gd/article/details/51055589
 
- win10 uwp json http://blog.csdn.net/lindexi_gd/article/details/51602405
 
- win10 uwp 从StorageFile获取文件大小 http://www.win10.me/?p=916
 
- win10 uwp 如何让WebView标识win10手机 http://www.win10.me/?p=914

- [win10 uwp 语音](win10_uwp_yu_yin.md)

- [win10 UWP 九幽数据分析](win10 UWP 九幽数据分析.md)
 
有时候我们会使用网络，本来我想写网络编程，现在没写，网络只有 http://blog.csdn.net/lindexi_gd/article/details/50838740

程序一般不是我们写完就运行，一般需要http://blog.csdn.net/lindexi_gd/article/details/50707981


然后我们可以看微软教程 国内最好的几个视频有[刘老师](http://www.win10.me/?cat=5)和[bilibili](http://space.bilibili.com/18340402)

如果要学比较高级的可以看https://github.com/Microsoft/Windows-universal-samples ，如果是开始还不懂使用，可以看：http://blog.csdn.net/bending1218/article/details/50523243

我将告诉大家一些案例，先一个有源代码的Markdown http://blog.csdn.net/lindexi_gd/article/details/50488191 

还有一个很简单win10 uwp 装机必备应用 含源代码 http://blog.csdn.net/lindexi_gd/article/details/50166161

使用Path win10 uwp 分治法 http://blog.csdn.net/lindexi_gd/article/details/51315914

移植 http://blog.csdn.net/lindexi_gd/article/details/51321064

关于网络一个完整可以 StreamSocket聊天室 http://www.wangchenran.com/uwp-streamsocket-chatroom-1.html

开发过程中我遇到一些诡异，发在下面：

- win10 uwp 无法附加到CoreCLR http://blog.csdn.net/lindexi_gd/article/details/51762068

- win10 uwp BadgeLogo 颜色 http://blog.csdn.net/lindexi_gd/article/details/51761788

- win10 输入法禁用IME http://blog.csdn.net/lindexi_gd/article/details/50117909

- win10 uwp 读取文本ASCII错误 http://www.win10.me/?p=938

当然我们也可以使用js来写，js参见：http://lib.csdn.net/article/csharp/32749 这个就是简单的使用js来写。

我们在开发会发现一些诡异的，或有些还是不懂，因为中文的书很少，但是我们可以看博客，我收很多人的博客，参见：http://blog.csdn.net/lindexi_gd/article/details/50032135

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  


