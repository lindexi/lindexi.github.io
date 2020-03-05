# 通过 frp 开启服务器打开本地的 ZeroNet 服务器外网访问

现在大多数的网站都需要有服务器，但是 ZeroNet 是无服务器的网络，可以在 ZeroNet 里面是否简单搭建自己的网站，同时不需要任何一个服务器。任何访问你的网站的小伙伴都将成为你的网站的服务器，每个人都可发布自己的网站而不需要服务器

<!--more-->
<!-- CreateTime:2019/3/9 11:47:48 -->

<!-- csdn -->

在中文译名是 零网 的 ZeroNet 是一个开源项目，是一个以对等网络用户为基础构成的类互联网的分布式网络，源代码在 [github](https://github.com/HelloZeroNet/ZeroNet) 可以找到

从 [github](https://github.com/HelloZeroNet/ZeroNet) 里面可以找到[中文介绍](https://github.com/HelloZeroNet/ZeroNet/blob/master/README-zh-cn.md) 里面详细告诉大家如何安装和使用

但是默认只能在本地访问，如在安装之后可以点击 [http://127.0.0.1:43110/1HeLLo4uzjaLetFx6NH3PMwFP3qbRbTf3D](http://127.0.0.1:43110/1HeLLo4uzjaLetFx6NH3PMwFP3qbRbTf3D) 访问 ZeroNet 首页

如果想要在手机端访问是很难的，但是如果有一个自己的服务器，可以通过 [frp](https://github.com/fatedier/frp/blob/master/README_zh.md) 的方法开启服务器转发本地的 ZeroNet 服务器

这样小伙伴通过访问你的服务器，就可以访问到你本地的 ZeroNet 网络，这样小伙伴就不需要在自己的设备上安装 ZeroNet 这样就可以愉快在手机上访问 ZeroNet 内容啦

下面一步步告诉大家如何通过服务器开启 frp 转发本地的 ZeroNet 网络

在开始阅读之前，要求你有下面的资源

- 一台有外网 ip 的服务器

- 有自己的域名

在服务器开启 frp 的方法请看[frp 官方文档](https://github.com/fatedier/frp/blob/master/README_zh.md) 

建议的服务器端配置是打开 frps.ini 修改里面的代码

```csharp
[common]
bind_port = 7000
vhost_http_port = 8080
subdomain_host = 你的二级域名
```

注意本文提供的方法要求你有一个域名，配置自己的二级域名的 A 记录到你的服务器地址，同时里面的记录需要是 `*.二级域名.域名.com` 如 `*.serivce.lindexi.com` 注意加上了 `*.` 就可以在服务器使用三级域名

这个方法可以让本地的 ZeroNet 作为三级域名

关于上面服务器的配置内容请看[frp 官方文档](https://github.com/fatedier/frp/blob/master/README_zh.md) 

在客户端的 frpc.ini 修改里面代码

```csharp
[common]
server_addr = 你的服务器的 Ip 如 123.123.123.123
server_port = 7000 # 这里对应服务器的 bind_port 的值

[zero] # 这个值表示的是客户端的命名，可以随意命名，推荐只使用英文小写字符
type = http 
local_port = 43110 # 本地的端口
local_ip = 127.0.0.1 # 本地的 ip 地址
subdomain = zero # 这就是定义三级域名是什么，如在 DNS 里面配置了 *.serivce.lindexi.com 到你的服务器 ip 那么现在通过 zero.serivce.lindexi.com 就可以访问到你的本文的 127.0.0.1:43110 的服务器
```

本地的 ZeroNet 默认开启的端口就是 43100 于是在客户端设置  subdomain 为 zero 就可以通过 zero.serivce.lindexi.com:8080 访问到本地的 127.0.0.1:43110 也就是小伙伴通过域名访问相当于他代理访问到你本地的网站

注意，如果有多个客户端，请修改客户端的命名，官网的文档里面默认写的是 `[web]` 如果有同时多个客户端连接，就会出现下面提示

```csharp
[W] [control.go:141] [web] start error: proxy name [web] is already in use
```

解决方法就是将 `[web]` 改为自己需要的命名

现在先开启 frp 的服务器端和客户端，然后开始配置 ZeroNet 啦

本地双击打开 ZeroNet 然后尝试访问一下你的配置域名，看 frp 是否成功配置，如果看到了网站显示下面内容，那么 frp 已经完成配置

```csharp
Forbidden. Invalid host
```

打开 zeronet.conf 文件修改内容

```csharp
[global]
fileserver_port = 25823
ui_ip = 0.0.0.0
ui_port = 43110
ui_host = 
 你的域名:对应 frps.ini 的 vhost_http_port 的端口
 127.0.0.1:43110
 localhost:43110
```

如我的域名是 `zero.service.lindexi.com` 默认开启端口是 8080 那么可以这样写

```csharp
[global]
fileserver_port = 25823
ui_ip = 0.0.0.0
ui_port = 43110
ui_host = 
 zero.service.lindexi.com:8080
 127.0.0.1:43110
 localhost:43110
```

现在我配置完成的是[http://zero.service.walterlv.com:8080](http://zero.service.walterlv.com:8080) 欢迎大家访问

下面有一些内容，只能在本地存在 ZeroNet 服务才能访问

[搭建Https的ZeroNet节点反向代理](http://127.0.0.1:43110/1FjA71G8f9vBkZcpL31vbijR6fq2ybXLEf/?Post:2:%E6%90%AD%E5%BB%BAHttps%E7%9A%84ZeroNet%E8%8A%82%E7%82%B9%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
