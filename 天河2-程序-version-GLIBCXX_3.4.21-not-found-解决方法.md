
# 天河2 程序 version GLIBCXX_3.4.21 not found 解决方法

本文告诉大家在 天河2 运行程序时发现 version GLIBCXX_3.4.21 not found 如何修复

<!--more-->



我在天河2运行一个程序报错

```
 version `GLIBCXX_3.4.21' not found
```

解决简单，在 [http://ftp.de.debian.org/debian/pool/main/g](http://ftp.de.debian.org/debian/pool/main/g)  找GCC，我现在是GCC6，可以下载更改的版本，下面的链接就是 GCC6 的链接

下载：[http://ftp.de.debian.org/debian/pool/main/g/gcc-6/libstdc++6_6.1.1-11_amd64.deb](http://ftp.de.debian.org/debian/pool/main/g/gcc-6/libstdc++6_6.1.1-11_amd64.deb)

[http://ftp.de.debian.org/debian/pool/main/g/gcc-4.9/libstdc++6_4.9.2-10_amd64.deb](http://ftp.de.debian.org/debian/pool/main/g/gcc-4.9/libstdc++6_4.9.2-10_amd64.deb)

可以进入 [http://ftp.de.debian.org/debian/pool/main/g/gcc-6/](http://ftp.de.debian.org/debian/pool/main/g/gcc-6/) 搜索`libstdc++6_`找到一个AMD64或I386

```

  ar -x libstdc++-6-dev_6.1.1-11_amd64.deb  && tar xvf data.tar.xz

```




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。