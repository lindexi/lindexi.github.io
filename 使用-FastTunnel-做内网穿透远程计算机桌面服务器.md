
# 使用 FastTunnel 做内网穿透远程计算机桌面服务器

FastTunnel是一款高性能跨平台内网穿透工具，使用它可以实现将内网服务暴露到公网供自己或任何人访问。对比市面上的其他内网穿透工具最大优势在于基于 dotnet 编写，提供二次开发扩展能力。本文将来告诉大家，在不做任何编程的情况下，通过 FastTunnel 工具，让我某台内网的 Windows 电脑，开启远程桌面服务，提供外网连入的能力

<!--more-->


<!-- CreateTime:2021/7/5 8:27:07 -->

<!-- 发布 -->

本文的演示需要三台计算机设备，其中一台是服务器，服务器拥有公网 ip 地址。如腾讯云的服务器。不挑服务器的系统，基于 dotnet 的 FastTunnel 轻松做到了在各个系统平台上运行

其中两台是私用的电脑，分别记为 公司的 和 家里的 电脑。这两台电脑都在不同的局域网里面。我的需求是让我可以在公司里面，通过远程桌面的方式，让公司的电脑可以远程到家里的电脑

此时我家里的电脑是没有对外有固定 IP 地址的，因此也就不能被公司的电脑直接访问。好在我有一台腾讯云的服务器，我准备通过 FastTunnel 工具，让腾讯云的服务器作为跳板，让我家里的电脑和公司的电脑可以连接

以下是实现方法：

先从 [https://github.com/FastTunnel/FastTunnel/releases](https://github.com/FastTunnel/FastTunnel/releases) 下载最新的版本

部署方式如官网所说，假定都是 Windows 系统，包括服务器和客户端。那么只需在服务器上下载解压缩 FastTunnel.Server 即可。在家里的电脑下载解压缩 FastTunnel.Client 即可。而公司的电脑啥用不用做哈

先在服务器端做一点配置，打开服务器端的 appsettings.json 文件，这就是服务器端的配置文件

```json
{
  "Logging": 
  {
    "LogLevel": 
    {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "ServerSettings": 
  {
    "BindAddr": "0.0.0.0", // 修改为 0.0.0.0 表示监听所有 ip 对应端口
    "BindPort": 10092, // 这是用来给 FastTunnel 连接的端口

    "Domain": "FastTunnel.lindexi.com", // 自定义域名web穿透必须

    // 服务监听的端口号, 访问自定义域名站点时url为 http://{SubDomain}.{Domain}:{ProxyPort_HTTP}/
    "ProxyPort_HTTP": 1270, // web穿透必须
    "HasNginxProxy": false // 可选，ngixn反向代理后可省略域名后的端口号进行访问
  }
}
```

以上是我的配置，我更改的部分就是 BindAddr 和 BindPort 两个值，还有自定义域名。此时我的腾讯云服务器将开放 10092 端口作为给 FastTunnel 客户端连接的端口。按照 FastTunnel 的设计，可以在客户端要求服务器端开放任意的其他端口给某个客户端使用

配置完成之后，双击 FastTunnel.Server 即可运行，如看到大概以下的内容就是运行成功

```
2021/07/04 09:12:31.123|DEBUG|===== FastTunnel Server Start =====
2021/07/04 09:12:31.278|DEBUG|FastTunnel Server Start
2021/07/04 09:12:31.305|DEBUG|????? -> 0.0.0.0:10092
```

接下来是来配置客户端部分，打开家里的电脑的 FastTunnel 客户端的 appsettings.json 文件，因为咱只是为了配置远程桌面内网穿透而已，因此只需要修改服务器端公网 ip 和服务器端通信端口，和 windows远程桌面 的内容。如下面代码，需要更改部分我标记出来

```json
{
  "Logging": 
  {
    "LogLevel": 
    {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "ClientSettings": 
  {
    "Common": 
    {
      // 【需要更改】
      // 服务端公网ip, 对应服务端配置文件的 BindAddr
      "ServerAddr": "14.106.92.12",

      // 【需要更改】
      // 服务端通信端口，对应服务端配置文件的 BindPort
      "ServerPort": 10092
    },
    "Webs": 
    [
      {
        // 本地站点所在内网的ip
        "LocalIp": "127.0.0.1",

        // 站点监听的端口号
        "LocalPort": 80,

        // 子域名, 访问本站点时的url为 http://{SubDomain}.{Domain}:{ProxyPort_HTTP}/
        "SubDomain": "test" // test.test.cc
      }
    ],

    /**
     * ssh穿透，ssh访问内网主机
     * 访问方式 #ssh -oPort=12701 {root}@{ServerAddr}
     * ServerAddr 填入服务端ip，root对应内网用户名
     */
    "SSH": 
    [
      {
        "LocalIp": "127.0.0.1",
        "LocalPort": 22,
        "RemotePort": 1273
      },
      {
        // 【需要更改】
        // 将本机的远程桌面开放
        "LocalIp": "127.0.0.1",
        "LocalPort": 3389, // windows远程桌面端口为3389
        // 【需要更改】
        // 让其他人通过服务器端的 RemotePort 端口，连接到 LocalIp 的 LocalPort 端口
        // 也就是说将 ServerAddr:RemotePort 映射到 LocalIp:LocalPort 地址
        "RemotePort": 10090
      }
    ]
  }
}
```

先修改服务端的 ip 地址，我的腾讯云的 ip 地址是 14.106.92.12 请将此值替换为你自己的服务器的 ip 地址。接着我在上面服务器端的配置里面写了 FastTunnel 开放的端口是 10092 因此需要在客户端设置和服务器端相同的值。从这里可以看到，服务器端设置的端口不是说给对外开放的，而只是给 FastTunnel 客户端用来连接使用的而已。接着为了设置客户端的远程桌面功能，还需要额外配置将本机的远程桌面开放

默认 Windows 的远程桌面地址就是本机的 3389 端口，因此可以做如上配置

接下来是在客户端配置连接到服务器端的哪个端口，也就是说服务器端对外开放的端口是由客户端决定的。如上面代码我设置了使用 10090 端口

以上配置的实现就是将 ServerAddr:RemotePort 映射到 LocalIp:LocalPort 地址，如上面配置是将 14.106.92.12:10090 映射到相对本机的 127.0.0.1:3389 端口

双击运行 FastTunnel.Client.exe 然后即可将家里的电脑的远程桌面开放出去。在公司的电脑可以通过远程桌面连接 14.106.92.12:10090 而远程到家里的电脑

使用起来非常方便，而且传输性能非常棒。更好的是 FastTunnel 是在 GitHub 和 Gitee 上完全开源的，请看 [https://gitee.com/Hgui/FastTunnel](https://gitee.com/Hgui/FastTunnel)

除了用来开放远程桌面外，还可以使用 FastTunnel 将内网的服务开放出去

更多请看 [FastTunnel-开源内网穿透框架 - gui.h - 博客园](https://www.cnblogs.com/springhgui/p/15005329.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。