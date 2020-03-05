
# 3分钟教你搭建 gitea 在 Centos 服务器

本文告诉大家如何在一个 Centos 服务器上搭建 gitea 然后在 gitea 创建帐号上传代码

<!--more-->



在开始之前简单介绍一下 gitea 这是一个轻量级的代码托管解决方案，代码开源，适合在自己的服务器搭建。

首先是创建一个存放 gitea 的文件夹，通过下面的命令可以创建一个空白的文件夹

> `cd ~`
> `mkdir gitea`

然后进入自己创建的文件夹

> `cd gitea`

下载最新的 gitea ，可以在这个网页找到最新的软件 https://docs.gitea.io/en-us/install-from-binary/

使用下面的代码将软件下载在文件夹

> `wget -O gitea https://dl.gitea.io/gitea/1.5.0/gitea-1.5.0-linux-amd64`

在运行软件之前需要先给权限

> `chmod +x gitea`

使用下面的命令可以运行 gitea 但是这个命令会在自己退出登录的时候自动关闭 gitea 如果需要一直在后台开启请看下面的命令

> `./gitea web`

很多时候在出现的问题都是因为没有安装好 git 通过下面的命令可以在 Centos 安装 git ，注意需要当前用户是管理员用户

> `yum install git`

如果可以运行，就在来浏览器输入服务器的ip和端口3000访问，我的服务器是 http://172.96.218.156 我可以使用下面的链接访问

http://172.96.218.156:3000

这个链接是可以使用的，大家可以使用我的服务器

现在就是做一个基础的设置，推荐使用 Sqlite 这样就不需要做配置

配置之后点击注册，第一个注册的用户就是管理员

如果需要让 gitea 在后台运行，可以使用下面的命令

> `nohup ./gitea web &`

这样退出登录在后台运行 gitea 可以在很多地方能访问到自己服务器就可以使用搭建的 gitea 另外 gitea 可以设置很多私有项目，这样就可以在服务器放一些不公开的项目

但是 gitea 的安全不是很好，还是不要在公开的服务器放自己重要的项目




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。