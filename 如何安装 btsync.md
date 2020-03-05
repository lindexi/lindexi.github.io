# 如何安装 btsync

本文告诉大家如何在 windows 和 Linux 安装使用 [Btsync](https://www.resilio.com) 而且分享一些小东西给大家

<!--more-->
<!-- CreateTime:2018/10/8 9:15:06 -->

<div id="toc"></div>

<!-- 标签：btsync -->

btsync 是分布式网盘
在这高速运作的信息化时代，使用云端来衔接工作和生活的点滴已是寻常事。可你是否曾扪心自问过：用各大云端备份自己的信息资料，真的安全放心吗？

毫不夸张的说，其实恶意代码和漏洞早已和你如影随形。你甚至都不用反驳说这是阴谋论，不信你去看看各网盘的「用户条款」和「隐私政策」。

所以一个可以放在本地的网盘是我们想要的

## 优点

btsync 把自己的电脑作为网盘，不限空间流量，适合局域网同步。

1. 不需要有自己的服务器

2. 不需要有公网 IP——如果两台设备都在【内网】，只要这两台设备都能访问到公网，就可以相互同步

3. 文件数量【无】限制，文件大小【无】限制

4. 支持多种网络形态——可以“公网上互相同步”，也可以是“局域网内相互同步”。

5. 【没有】审查

现在 Btsync 也叫 resilio 同步工具

## windows 安装

如何使用？

首先下载安装，然后随意给一个名称。[点击下载 密码: ns6u](链接: https://pan.baidu.com/s/1c1CFkw4 )，如果百度无法下载，可以到[这里](http://download.csdn.net/detail/lindexi_gd/9792343)，下载如果无法下载可以联系我

官网是 [https://www.resilio.com](https://www.resilio.com/platforms/desktop/)

[Resilio-Sync btsync x64 2.6.1-CSDN下载](https://download.csdn.net/download/lindexi_gd/10705179 )

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F201732412536.jpg)

我接下来分享的都会在 btsync ，那么如何分享？

需要拖入一个文件夹，可以看到自动出现文件秘钥

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017324125426.jpg)

复制只读秘钥发给朋友，他就可以使用。

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017324125512.jpg)

那么拿到秘钥如何获得资源？

点击手动链接

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017324125540.jpg)


输入秘钥

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017324125612.jpg)

新建文件夹，之后的文件就会放在这个文件夹

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017324125659.jpg)

一个秘钥一般需要新建一个文件夹。

点击确定就好了，这样就获得朋友的文件。

## Centos 6 安装

本文告诉大家如何安装特点的 1.4 版本。

首先上传解压的文件，我使用的是 btsync_i386-1.4.111 ，直接上传到服务器任意的文件夹

然后使用下面的代码解压

```csharp
tar xvf btsync_i386-1.4.111.tar.gz
```

于是文件 btsync 就解压出来了。如果运行出现下面的问题，那么用 yum 就可以安装

```csharp
 /lib/ld-linux.so.2: bad ELF interpreter: No such file or directory

```

安装的方法

```csharp
yum -y install glibc.i686

```

安装完成可以运行 btsync 不过需要获得配置，如果没配置就难以从网页打开

```csharp
./btsync --dump-sample-config > btsync.conf

```

然后使用 vi 就可以打开配置，可以修改配置的默认端口

```csharp
vi btsync.conf
```

可以看到下面的配置

```csharp
{
  "device_name": "My Sync Device",
  "listening_port" : 0, // 0 - randomize port 这是软件监听端口，不是网页的端口

/* storage_path dir contains auxilliary app files if no storage_path field: .sync dir created in the directory
   where binary is located. otherwise user-defined directory will be used */
// "storage_path" : "/home/user/.sync",

/* set location of pid file */
// "pid_file" : "/var/run/btsync/btsync.pid",

/* use UPnP for port mapping */
  "use_upnp" : true,

/* limits in kB/s. 0 - no limit */
  "download_limit" : 0,
  "upload_limit" : 0,

/* proxy configuration */
// "proxy_type" : "socks4", // Valid types: "socks4", "socks5", "http_connect". Any other value means no proxy
// "proxy_addr" : "192.168.1.2", // IP address of proxy server.
// "proxy_port" : 1080,
// "proxy_auth" : false, // Use authentication for proxy. Note: only username/password for socks5 (RFC 1929) is supported, and it is not really secure
// "proxy_username" : "user",
// "proxy_password" : "password",

  "webui" :
  {
    "listen" : "0.0.0.0:8888" // remove field to disable WebUI 修改这里可以打开网页，监听端口可以修改

/* preset credentials. Use password or password_hash */
  ,"login" : "admin"//网页需要添加账号密码，请自己设置
  ,"password" : "password"//这是密码
//  ,"password_hash" : "some_hash" // password hash in crypt(3) format
//  ,"allow_empty_password" : false // Defaults to true
/* ssl configuration */
//  ,"force_https" : true // disable http 如果需要使用 https 那么取消注释
//  ,"ssl_certificate" : "/path/to/cert.pem" 这时需要添加证书
//  ,"ssl_private_key" : "/path/to/private.key"

/* directory_root path defines where the WebUI Folder browser starts (linux only). Default value is / */
//  ,"directory_root" : "/home/user/MySharedFolders/"

/* dir_whitelist defines which directories can be shown to user or have folders added (linux only)
   relative paths are relative to directory_root setting */
//  ,"dir_whitelist" : [ "/home/user/MySharedFolders/personal", "work" ]
  }

/* !!! if you set shared folders in config file WebUI will be DISABLED !!!
   shared directories specified in config file  override the folders previously added from WebUI. */
/*, 如果删除注释，就不可以用网页，直接代码设置分享的文件夹
  "shared_folders" :
  [
    {
      "secret" : "MY_SECRET_1", // required field - use --generate-secret in command line to create new secret
      "dir" : "/home/user/bittorrent/sync_test", // * required field
      "use_relay_server" : true, //  use relay server when direct connection fails
      "use_tracker" : true,
      "use_dht" : false, //使用 dht，这个一般需要打开
      "search_lan" : true,//局域网
      "use_sync_trash" : true, // enable SyncArchive to store files deleted on remote devices
      "overwrite_changes" : false, // restore modified files to original version, ONLY for Read-Only folders
      "known_hosts" : // specify hosts to attempt connection without additional search
      [
        "192.168.1.2:123" //预定义主机
      ]
    }
  ]
*/

/* Advanced preferences can be added to config file. Info is available at http://sync-help.bittorrent.com */

}

```

## 国内如何下载

如果需要在国内使用，那么需要指定预定义主机。btsync可以使用dht进行下载，他的难就在于发现第一个节点。如果发现了一个节点，就可以通过他得到其他的节点。预定义主机就是自己已经知道存在的一个主机。

可以通过代理访问到外面的节点，然后保存他，之后不使用代理也可以下载。或者设置发现的主机，这样通过这个可以得到其它的节点。

首先创建一个文件夹，然后点击设置

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017111191010.jpg)

例如我添加字体文件，点击设置 打开dht和添加主机，这时不需要设置代理就可以下载，不过需要等很久

如果需要设置代理，那么需要注意默认用的是 socket4 需要设置为支持的代理。

参见：[BitTorrent Sync Installer 1.4.111](http://getos.org/btsync/14111.html )

https://program-think.blogspot.com/2015/01/BitTorrent-Sync.html?utm_source=tuicool&utm_medium=referral

## 搭建预定义主机

可以购买一个服务器，然后在服务器开启 btsync 这时记录他的 ip 和监听端口，监听端口是软件监听端口。于是这就是预定义主机，需要在他这里同步一些文件，然后设置开启 dht ，这样可以让他去寻找一些节点。

在自己的电脑上，使用预定义主机，开启dht就可以从预定义主机拿到其他节点，所以就可以访问其它的资源。

如果是在校园网，可以通过 btsync 分享资源，做法是开放一台中转服务器，服务器是不关机的，或者基本都是开的，而且ip不会变化，如果ip会变化，就需要使用特殊方法。首先记录下自己的服务器ip地址和端口号，端口就是监听端口，然后把这个告诉大家，让大家添加预定义主机。这样就可以通过这个获取到所有人的 btsync ，通过这个就可以传输资源。

如果 ip 会变化，那么可以使用 meibu 提供的方法，请看 [http://meibu.com/](http://meibu.com/ )，注册之后就可以使用域名，于是用域名的方法设置服务器就好啦。

如果发现你的局域网无法使用，欢迎联系我。 

## ZeroTier

最简单的方式是使用 ZeroTier ，如何使用请看

[ZeroTier – 无配置，零基础「内网穿透」随时随地连回家/学校/办公室 [跨平台] - 小众软件](https://www.appinn.com/zerotier-one/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  