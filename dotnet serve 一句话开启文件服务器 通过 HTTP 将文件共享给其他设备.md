# dotnet serve 一句话开启文件服务器 通过 HTTP 将文件共享给其他设备

在当前的 IT 领域，我推荐每个开发设备都应该安装 dotnet 这个工具，在 dotnet 这个工具上有大量开发者提供了无数好用的工具。本文要告诉小伙伴的工具是 natemcmaster 大佬提供了 serve 工具，可以用来开启本地文件服务器，使用非常简单。这个方案适合在 Windows 系统和 Linux 系统和 MAC 上使用，发布的 HTTP 服务可以在任何能访问到的设备上使用

<!--more-->
<!-- CreateTime:2020/2/4 17:30:05 -->

<!-- 发布 -->

默认小伙伴电脑已经安装了 [dotnet](https://dotnet.microsoft.com/) 工具，如果还没有安装，请到[官网](https://dotnet.microsoft.com/)下载安装

首次使用需要安装，请打开 cmd 控制台输入下面代码安装

```
dotnet tool install --global dotnet-serve
```

安装完成之后通过 cmd 控制台进入到需要发布共享的文件夹里面，如我需要发布 `F:\lindexi\foo.txt` 这个资源到我的笔记本 ubuntu 系统，我先进入到 `F:\lindexi` 文件夹

```
cd /d F:\lindexi
```

然后就是一句话开启文件服务器了

```
dotnet serve -p 1007
```

默认会使用 8080 端口，上面代码的 -p 就是指定端口为 1007 如果接受默认端口，那么可以使用 `dotnet serve` 开启服务器

此时用浏览器访问 `http://127.0.0.1:1007` 就可以看到文件了

等等，我还有个问题，我不想用 8080 端口，我也不想去想一个端口，毕竟我不知道哪个端口可以用，可以使用 `-p 0` 这样就能自动给一个随机端口

我还有一个问题，为什么在我的另一个设备上访问 ip 的方式拿不到？原因是上面的命令开启的是 localhost 的监听，想要让其他设备也能访问到，可以使用下面命令

```
dotnet serve -p 0 -a 0.0.0.0
```

上面代码的 `-p 0` 有仔细看文章的小伙伴就知道是用来做什么的，后续的 `-a 0.0.0.0` 的意思就是监听本地的所有 IP 地址，此时通过 `ipconfig` 或 Linux 下的 `ifconfig` 可以看到本地所在局域网内的 ip 地址，让其他设备通过 `http://ip:端口` 在浏览器打开就可以访问到了

在 Linux 下可以通过 wget 命令输入链接下载到文件，如我的设备上可以通过下面代码 `wget -nc htp://172.18.134.16:1007/foo.txt` 拿到我另一个设备的文件

那我不想要开启服务器了可以如何关闭？其实运行时有提示按下 ctrl+C 也就是复制键就可以关闭服务了，或者关闭控制台也可以关闭服务

这个命令工具特别适合用来暂时开启本地文件的共享，当然也适合用来做二次开发，用其他弱功能的脚本调用命令行让弱功能的脚本能做到开启本地服务

如果小伙伴想要了解这个工具是如何做的，请看[源代码](https://github.com/natemcmaster/dotnet-serve) 如果访问不了，那么请到 [gitee.com](https://gitee.com/ ) 注册帐号点击新建项目，粘贴 `https://github.com/natemcmaster/dotnet-serve` 就可以将 github 的仓库同步到国内的 gitee 了

如果我还要让外网的其他小伙伴和我不在一个局域网的访问，可以怎么办？请看 [dotnet core 通过 frp 发布自己的网站](https://blog.lindexi.com/post/dotnet-core-%E9%80%9A%E8%BF%87-frp-%E5%8F%91%E5%B8%83%E8%87%AA%E5%B7%B1%E7%9A%84%E7%BD%91%E7%AB%99.html) 

在外网发布了，我就想使用 gzip 减少内容传输，此时可以添加 `-z` 命令

这个工具也适合前端的小伙伴开启本地服务器，看自己写的前端代码

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
