# 天河2 程序 version GLIBCXX_3.4.21 not found 解决方法

本文告诉大家在 天河2 运行程序时发现 version GLIBCXX_3.4.21 not found 如何修复

<!--more-->
<!-- CreateTime:2019/6/23 11:51:16 -->


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