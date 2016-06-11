# win10 uwp win2d

Win2d

##介绍

Win2d是一个很简单使用的底层图形Windows Runtime API，可以使用硬件加速，主要是GPU的强大计算。他可以使用C#或C++写应用商店应用，包括UWP或windows 8.1手机或电脑。他利用强大的Direct2D，无缝集合windows的Xaml，可以使用强大的渲染得到漂亮界面。

使用他可以将界面交给GPU，让CPU集中计算我们的算法

我们可以通过Nuget来得到win2d，Nuget的windows10版win2d:http://www.nuget.org/packages/Win2D.uwp，Nuget的windows 8.1版win2d:http://www.nuget.org/packages/Win2D.win81

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

- 图形显影效应
