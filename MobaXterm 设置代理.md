# MobaXterm 设置代理

有时候访问的服务器是需要代理才可以访问，本文告诉大家如何在 ssh 连接使用代理

<!--more-->
<!-- csdn -->

<!-- 标签 ：MobaXterm，代理，ssh，服务器，proxy，vps,软件 -->

在开始本文之前，需要大家已经设置好代理

## ssh 连接

最简单的方法是在命令输入 ssh 加上代理，不过这个方法容易失败

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018213145431.jpg)

```csharp
ssh -D 代理端口 连接的服务器
```

如果需要指定端口，请加上 `-p 端口`

## 通道

另一个方法是使用通道，点击 tool 可以看到 ssh 通道，打开之后可以看到这个界面。

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201821314568.jpg)

点击 创建通道，然后输入服务器和端口

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018213145618.jpg)

需要选择动态端口，然后才可以看到这个界面

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018213145656.jpg)

设置完成点保存，然后开启

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018213145741.jpg)

## 连接代理

上面的两个方法都是比较复杂，那么告诉大家一个最简单的方法

右击点击编辑连接，可以看到这个界面

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201821314599.jpg)

选择 Use Proxy 选择 Socks5 然后加上代理服务器就可以了。

## 中转

如果有中转的服务器，那么也可以在下面这个界面选择 Connect through ssh gateway

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201821314599.jpg)

然后输入连接的服务器地址和用户名

如果大家在使用的时候发现任何问题，欢迎联系我

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
