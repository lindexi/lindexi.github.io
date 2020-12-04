
# dotnet 在 UOS 国产系统上使用 MonoDevelop 创建 GTK 全平台带界面应用

本文告诉大家如何在 UOS 国产系统上开始使用 MonoDevelop 开发，通过创建 GTK# 应用，进入界面开发的第一步

<!--more-->


<!-- CreateTime:2020/8/31 20:10:00 -->



在开始之前需要小伙伴先安装好 MonoDevelop 工具

安装完成之后，可以在开始菜单找到这个 MonoDevelop 工具

<!-- ![](image/dotnet 在 UOS 国产系统上安装 MonoDevelop 开发工具/dotnet 在 UOS 国产系统上安装 MonoDevelop 开发工具0.png) -->

![](http://image.acmx.xyz/lindexi%2F20208311957535612.jpg)

打开之后，点击新建项目

<!-- ![](image/dotnet 在 UOS 国产系统上使用 MonoDevelop 创建 GTK 全平台带界面应用/dotnet 在 UOS 国产系统上使用 MonoDevelop 创建 GTK 全平台带界面应用0.png) -->

![](http://image.acmx.xyz/lindexi%2F20208312011468845.jpg)

选择新建 GTK# 2.0 的项目，点击下一步，选择路径，加上项目名

请小伙伴记录这个路径，因为 UOS 的资源管理器做的比较弱，需要咱自己通过控制台去找到这个路径

其实MonoDevelop就是基于 GTK# 作为底层渲染的，因此能做到啥小伙伴大家看这个 IDE 也就有底了，有趣的是这个工具也是完全开源的

<!-- ![](image/dotnet 在 UOS 国产系统上使用 MonoDevelop 创建 GTK 全平台带界面应用/dotnet 在 UOS 国产系统上使用 MonoDevelop 创建 GTK 全平台带界面应用1.png) -->

![](http://image.acmx.xyz/lindexi%2F20208312012512841.jpg)

新建的项目默认啥都没有，可以按下 F5 开始执行

此时就完成了构建逻辑了

可以在命令行里面通过 `mono xx.exe` 执行构建出来的工具，当前只有一个空白窗口啥都没有，但是这是一个好的开始





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。