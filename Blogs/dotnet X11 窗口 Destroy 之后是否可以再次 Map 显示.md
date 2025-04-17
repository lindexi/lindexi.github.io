---
title: dotnet X11 窗口 Destroy 之后是否可以再次 Map 显示
description: 经过我的测试，在 X11 里面，如果先使用 XDestroyWindow 销毁窗口，将不能再次使用 XMapWindow 显示窗口
tags: dotnet,X11
category: 
---

<!-- CreateTime:2024/08/14 07:27:21 -->

<!-- 发布 -->
<!-- 博客 -->

简单的测试代码如下

```csharp
bool destroyWindow = false;

            if (!destroyWindow)
            {
                XDestroyWindow(display, handle);
                XFlush(display);
                destroyWindow = true;
            }
            else
            {
                XMapWindow(display, handle);
                XFlush(display);
            }
```

上述代码的 display 是通过 XOpenDisplay 函数的返回值获取的，上述代码的 handle 是通过 XCreateWindow 函数的返回值获取的，是 X11 窗口的 XID 的值

运行以上代码，先进入 `XDestroyWindow(display, handle)` 然后再次运行执行 `XMapWindow(display, handle)` 将看到大概如下的错误输出

```
X Error of failed request:  BadWindow (invalid Window parameter)
  Major opcode of failed request:  25 (X_SendEvent)
  Resource id in failed request:  0x9800002
  Serial number of failed request:  13
  Current serial number in output stream:  13
```

通过以上代码可以证明窗口 XDestroyWindow 之后，不可以再次调用 XMapWindow 显示窗口。如果只是想隐藏窗口，可以使用 XUnmapWindow 函数进行隐藏窗口

这也就是为什么大部分[教程](http://www.cppblog.com/zmj/archive/2007/05/18/24331.html)都只在清理时才调用 XDestroyWindow 方法的原因

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/a7a1054b3b1913cadd08b01547f808f9500dc5bb/X11/GaroceabairCibiwhemfi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/a7a1054b3b1913cadd08b01547f808f9500dc5bb/X11/GaroceabairCibiwhemfi) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin a7a1054b3b1913cadd08b01547f808f9500dc5bb
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin a7a1054b3b1913cadd08b01547f808f9500dc5bb
```

获取代码之后，进入 X11/GaroceabairCibiwhemfi 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
