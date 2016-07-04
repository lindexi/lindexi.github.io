# win10 uwp win2d

Win2d

本文主要翻译，可能带有一定的主观性和局限性，说的东西可能不对或者不符合每个人的预期。如果觉得我有讲的不对的，就多多包含，或者直接关掉这篇文章，但是请勿生气或者发怒吐槽，可以在我博客评论 http://blog.csdn.net/lindexi_gd

##介绍

Win2d是一个很简单使用的底层图形Windows Runtime API，可以使用硬件加速，主要是GPU的强大计算。他可以使用C#或C++写应用商店应用，包括UWP或windows 8.1手机或电脑。他利用强大的Direct2D，无缝集合windows的Xaml，可以使用强大的渲染得到漂亮界面。

使用他可以将界面交给GPU，让CPU集中计算我们的算法

我们可以通过Nuget来得到win2d，Nuget的windows10版win2d:[http://www.nuget.org/packages/Win2D.uwp](http://www.nuget.org/packages/Win2D.uwp)，Nuget的windows 8.1版win2d:http://www.nuget.org/packages/Win2D.win81

如何使用可以参见微软示例http://github.com/Microsoft/Win2D-samples

在下面我们会说如何快速使用。

一些链接：
如果找到bug可以通过 http://github.com/Microsoft/Win2D/issues
团队博客：http://blogs.msdn.com/b/win2d


##特性

- 位映像图形
 
 - 加载、保存、渲染图形
  
 - 纹理渲染
 
 - 蒙版
 
 - 快速处理大量图片
 
 - 分块压缩图像
 
 - 加载、保存、渲染虚拟位图，虚拟位图就是超过GPU的纹理会自动分为多个

- 矢量图

 - 画图形，线、矩形、圆，或使用基础图形组成复杂的
 
 - 使用笔刷、颜色、图形填充图形
 
 - 任意宽度线段


- 图形显影效应


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
