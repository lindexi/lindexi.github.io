# 微软的 P2P 下载方式

微软在 2016 的时候就说要做 P2P 提供的更新，因为微软说每次系统更新使用的服务器费用很大，同时很多用户都说连不上微软的服务器。但是很快微软就砍了这个技术，原因是P2P的水很深。不过微软收购了 Pando Networks 公司，这是专业做 NYC-based P2P 技术的公司，后续微软就在更新和商店下载使用了P2P技术

<!--more-->
<!-- CreateTime:2019/9/27 9:44:44 -->

<!-- csdn -->

现在在应用商店下载和系统的更新走的都是微软的 P2P 下载方式，通过 P2P 下载方式不仅可以帮微软省很多服务器费用，同时也能让用户的下载速度快很多，特别是局域网

打开任务管理器，看一下 Background Intelligent Transfer Service 服务是不是占用了很多的 CPU 如果是那么证明系统在进行 P2P 的上传或下载

作为开发，我关注的是微软正在使用 P2P 下载什么内容，打开 PowerShell 输入下面命令

```csharp
PS  Get-DeliveryOptimizationStatus
```

如果此时在进行上传或下载，那么将会显示下面差不多的内容

```
FileId                      : 46de28dd16e575167f79299f5bffa163a8f10266
FileSize                    : 52978067
TotalBytesDownloaded        : 41443731
PercentPeerCaching          : 100
BytesFromPeers              : 41443731
BytesFromHttp               : 0
Status                      : Caching
Priority                    : Background
BytesFromCacheServer        : 0
BytesFromLanPeers           : 40395155
BytesFromGroupPeers         : 0
BytesFromInternetPeers      : 1048576
BytesToLanPeers             : 80790310
BytesToGroupPeers           : 0
BytesToInternetPeers        : 14352384
DownloadDuration            : 00:00:18.7020000
HttpConnectionCount         : 0
LanConnectionCount          : 7
GroupConnectionCount        : 0
InternetConnectionCount     : 128
DownloadMode                : 3
SourceURL                   : http://tlu.dl.delivery.mp.microsoft.com/filestreamingservice/files/20852e53-5ebc-49f2-bfc
                              1-1c032251c75e?P1=1569511260&P2=402&P3=2&P4=iQHSHSFDobj/rWWd1RT/ln/38wPW6hNrkyk+pwU8bp6CE
                              lL6E5/RNVGM8ZoCWq5WjQCwpjUlfqH+gak8Fj+wiw==
NumPeers                    : 290
PredefinedCallerApplication : WU Client Download
ExpireOn                    : 2019/9/29 22:55:18
IsPinned                    : False
```

对应的关键属性如下

- FileId 说明下载的是什么文件
- FileSize 文件的大小
- TotalBytesDownloaded 总下载大小
- BytesFromPeers 从 P2P 下载的文件大小
- BytesFromHttp 从 HTTP 下载的文件大小，也就是从微软服务器下载的大小是多少
- BytesFromLanPeers 从局域网的下载的文件大小
- DownloadMode 0 仅从HTTP下载，1 从局域网下载，2 从 Group 下载，3 从 Internet 的其他P2P设备下载
- BytesFromInternetPeers 从外网的P2P设备下载的文件大小
- BytesToLanPeers 传给局域网设备的文件大小
- BytesToInternetPeers 传给外网P2P设备的文件大小

通过对比 BytesFromPeers 和 BytesFromHttp 的大小就可以知道使用了 P2P 可以给微软节省了多少服务器费用，虽然微软有Azure可以使用空闲服务器做系统升级等，所以更多看重的是速度的提升

从上面的数据可以看到，这次下载的文件都是从 P2P 下载的，部分从外网的设备下载资源，更多的是从局域网下载的，此时的下载速度将会很快。同时没有从 HTTP 服务器下载资源，也就是这个资源的下载，微软只是告诉存在这个资源，资源的下载都是从P2P下载不占用任何微软的服务器

通过 `Get-DeliveryOptimizationPerfSnap -Verbose` 可以知道总的下载和上传文件大小

```csharp
FilesDownloaded                 : 2
FilesUploaded                   : 2
TotalBytesDownloaded            : 58,276,025
TotalBytesUploaded              : 137,054,360
AverageDownloadSize             : 29,138,012
AverageUploadSize               : 68,527,180
DownloadMode                    : 3
Files                           : 2
CacheSizeBytes                  : 58,276,025
TotalDiskBytes                  : 126,915,186,688
AvailableDiskBytes              : 9,873,575,936
NumberOfPeers                   : 518
CdnConnections                  : 7
LanConnections                  : 10
GroupConnections                : 0
InternetConnections             : 249
DownlinkBps                     : 35,382
UplinkBps                       : 6,834
ForegroundDownloadRatePct       : 90
BackgroundDownloadRatePct       : 45
UploadRatePct                   : 100
UploadCount                     : 2
```

在 1903 和以上的系统可以有更多的 PowerShell 命令用于控制 P2P 文件的分发，详细请看[官方文档](https://docs.microsoft.com/en-us/windows/deployment/update/waas-delivery-optimization-setup)

我找了很多文档，现在微软没有将P2P网络开放给开发者，同时限定了资源分发的域名。也就是自己的资源是无法接入到微软的P2P网络的

在 1511 以上的系统都默认开启了 P2P 功能，在世界上有很多电脑都会开启这个P2P功能，于是微软就搭建了世界上最大的P2P网络，如果能接入这个网络，那么网络发现等问题都可以让系统统一做，但是我认为如果微软开放了开发，那么将会很快被干掉，因为会存在大量版权问题，以及zz问题

使用P2P更新不仅可以省服务器也可以提高用户的下载速度，难道只有机智微软会这么做？其实微软是落后很久才做出来的，在谷歌浏览器的 Update Engine 就可以通过修改 device policy 开启P2P的功能。另一个大公司 Twitter 也使用了 P2P 做更新，不过不是更新客户端，而是更新服务器

[BitTorrent-like P2P software updates could be coming to Windows 10 - ExtremeTech](https://www.extremetech.com/computing/201269-bittorrent-like-p2p-software-updates-could-be-coming-to-windows-10 )

[http://patents.com/us-7639805.html](http://patents.com/us-7639805.html)

[Set up Delivery Optimization](https://docs.microsoft.com/en-us/windows/deployment/update/waas-delivery-optimization-setup )

[Configure Delivery Optimization for Windows 10 updates (Windows 10)](https://docs.microsoft.com/en-us/windows/deployment/update/waas-delivery-optimization )

[Twitter 使用P2P更新服务器](https://blog.twitter.com/engineering/en_us/a/2010/murder-fast-datacenter-code-deploys-using-bittorrent.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
