
# dotnet 进行二进制差分压缩文件

我需要对一个文件做二进制差分压缩，我有一个文件的起始点，在之后的每次更改我都记录文件的二进制的差分，这样就可以通过起始点和差分文件计算修改后的文件。通过二进制差分可以用来提高文件保存磁盘读写速度，也可以减少软件自动更新需要的文件大小

<!--more-->


<!-- CreateTime:2019/12/24 9:27:49 -->

<!-- csdn -->

<!-- 不发布 -->

在 Chrome 的软件更新就使用这个技术，通过二进制差分方法下载差分文件，然后用差分文件和当前版本计算出新版本


本文代码放在 [github](https://github.com/lindexi/lindexi_gd/blob/739bb867bd62d9356dc5a3d189e9e1d63daf4a69/LwufxgbaDljqkx/) 欢迎小伙伴访问




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。