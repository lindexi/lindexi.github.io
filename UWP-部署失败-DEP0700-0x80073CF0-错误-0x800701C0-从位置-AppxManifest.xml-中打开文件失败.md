
# UWP 部署失败 DEP0700 0x80073CF0 错误 0x800701C0 从位置 AppxManifest.xml 中打开文件失败

本文记录 UWP 应用程序部署失败，提示 0x80073CF0 错误 0x800701C0: 从位置 AppxManifest.xml 中打开文件失败，错误为无法遍历该路径，因为它包含不受信任的装入点

<!--more-->


<!-- CreateTime:2025/01/17 07:05:23 -->

<!-- 发布 -->
<!-- 博客 -->

具体的错误内容如下

DEP0700: 应用程序注册失败。[0x80073CF0] 错误 0x800701C0: 从位置 AppxManifest.xml 中打开文件失败，错误为: 无法遍历该路径，因为它包含不受信任的装入点。 

对应的英文版错误大概内容如下

DEP0700: Registration of the app failed. [0x80073CF0] error 0x800701C0: Opening file from location: AppxManifest.xml failed with error: The path cannot be traversed because it contains an untrusted mount point.

我尝试删除 bin 和 obj 文件夹，重新生成都没有用。重新新建空白 UWP 项目也没有用

解决方法

此问题出现的本质原因是我的代码文件夹是一个放在使用 mklink 创建的软链接文件夹里面，软链接内的 AppxManifest.xml 文件不在受信任范围内，导致注册失败

知道了这个原因之后，我就去掉了软链接，换成实际的文件夹。不在软链接的文件夹内创建项目。如此即可解决此问题




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。