# visual Studio 无法调试，提示程序跟踪已退出

今天在打码出现了vs无法调试，我在网上查了很久没有发现一个方法。

vs点击启动时，出现了一下提示


```csharp
  程序“[12648] *.vshost.exe: 程序跟踪”已退出，返回值为 0 (0x0)。
  程序“[12648] *.vshost.exe”已退出，返回值为 -1073741819 (0xc0000005) 'Access violation'。
```


<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>


这让我无法打码，于是发现同学的vs的设置和我的有那些不同，最后发现了，在工程属性，Debug页面里，没有勾选 `启用本地调试`。

如果是英文版，那么就是 Properties 上打开 Debug 页面，debug页面就是调试页面，勾选 `Enable unmanaged code debugging`

![这里写图片描述](http://img.blog.csdn.net/20170120093018668?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGluZGV4aV9nZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这样就可以。

参见：http://www.cnblogs.com/know-life-death/archive/2011/07/04/2097841.html

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  