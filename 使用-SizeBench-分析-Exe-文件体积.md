
# 使用 SizeBench 分析 Exe 文件体积

本文将介绍微软开源免费的 SizeBench 工具，使用 SizeBench 工具可以用来分析 Exe 二进制文件的体积，分析 Exe 文件大小里面有哪些是可以优化的

<!--more-->


<!-- 博客 -->
<!-- 发布 -->

下载安装方式：

请前往[应用商店](https://www.microsoft.com/store/productId/9NDF4N1WG7D6)安装，应用商店地址：[https://www.microsoft.com/store/productId/9NDF4N1WG7D6](https://www.microsoft.com/store/productId/9NDF4N1WG7D6)

工具的开源项目地址：[https://github.com/microsoft/SizeBench](https://github.com/microsoft/SizeBench)

使用方法：

安装完成之后，即可在开始菜单找到 SizeBench 应用，点击打开

当前的 SizeBench 工具提供两个功能，第一个就是分析 Exe 等 PE 文件的二进制体积，分析 PE 文件包含哪些内容，有哪些是重复的。第二个是对同一个 PE 文件的多个版本进行分析，了解多个版本之间的差异

本文着重介绍第一个功能，打开界面之后，点击 `Examine a binary` 然后选择 Exe 或 DLL 等 PE 文件和对应的 PDB 符号文件，如下图

<!-- ![](image/使用 SizeBench 分析 Exe 文件体积/使用 SizeBench 分析 Exe 文件体积0.png) -->

![](http://image.acmx.xyz/lindexi%2F202382844425092.jpg)

选择完成之后即可进入分析界面，分析界面的内容里面将会包含 PE 文件的各个部分的大小，比如包含的资源的大小，引用的静态库占用的大小。重复的字符串等内容的大小




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。