# win10 uwp 无法附加到CoreCLR


本文说的是在vs调试无法附加到CoreCLR。拒绝访问。已经如何去解决，可能带有一定的主观性和局限性，说的东西可能不对或者不符合每个人的预期。如果觉得我有讲的不对的，就多多包含，或者直接关掉这篇文章，但是请勿生气或者发怒吐槽，可以在我博客评论 http://blog.csdn.net/lindexi_gd
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

1. vs安装在C盘

 如果不是安装在C盘，把vs卸载，安装在C盘

2. 部署应用放在C盘

 ![这里写图片描述](http://img.blog.csdn.net/20160620090717158)

 放在E就错

 那么这个在哪，其实打开设置，进系统，存储

3. vs2015更新

  之前我的应用无法打包，最后升级vs就好啦，所以看是不是版本，因为vs没有和win10版本出现
  
1. 修复，打开安装包，修复

2. 关闭所有杀毒软件试试

接着还有工程文件放在C盘

不要修改临时变量文件

## 查询临时变量：

我的电脑，属性，高级

![这里写图片描述](http://img.blog.csdn.net/20160620091524560)
这样就是不对的，检查是原来的临时变量

临时变量无论是用户变量还是系统变量都需要为原来

如果使用我说的还是这样，最后用你的方式解决了，那么请告诉我你的方法，或自己写一篇博文来救我们这些渣渣

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

