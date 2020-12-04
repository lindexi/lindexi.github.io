
# dotnet 在国产 UOS 系统利用 dotnet tool 工具做文件传输

我在一台设备上安装了 UOS 系统，但是我如何在我的主开发设备上和 UOS 系统传输文件？通过 dotnet tool 工具可以完成大部分的工作，当然，使用 dotnet tool 不仅做文件传输，还能做很多特别强大的应用。本文就使用文件传输作为例子告诉大家如何使用 dotnet tool 在多个平台里面使用相同的一套技术和命令行作为工具

<!--more-->


<!-- CreateTime:2020/9/5 8:30:53 -->



在 dotnet 里，可以制作一个 dotnet 应用，将这个应用发布到 NuGet 上，无论是外网的 NuGet 的还是自己内网搭建的都可以，然后其他小伙伴就可以通过 NuGet 下载到这个应用。这样的应用就叫 dotnet tool 工具

使用 dotnet tool 工具的优势在于，工具使用自己熟悉的 .NET 编写，其次是分发方便，毕竟咱的项目基本上都需要安装 NuGet 包，也就是咱都是有 NuGet 源和环境的，因此可以方便在设备上利用 NuGet 获取工具或上传自己的工具

在开始之前，请在 UOS 上安装必要的工具，如 dotnet 开发工具，请看 [dotnet 在 UOS 国产系统上安装 dotnet sdk 的方法](https://blog.lindexi.com/post/dotnet-%E5%9C%A8-UOS-%E5%9B%BD%E4%BA%A7%E7%B3%BB%E7%BB%9F%E4%B8%8A%E5%AE%89%E8%A3%85-dotnet-sdk-%E7%9A%84%E6%96%B9%E6%B3%95.html )

接下来咱需要使用的工具有两个，一个是用来开启本地文件服务器的，另一个是用来下载文件的

在 UOS 上有一个限制，也就是不能调用全局的工具，只能使用文件夹内本地工具，这个问题我和官方说了，请看 [dotnet global tool can not run in UOS system · Issue #13399 · dotnet/sdk](https://github.com/dotnet/sdk/issues/13399 )

例如我期望在 UOS 上传输一个文件到我的开发机器上？可以如何做？可以使用下面步骤

1. 在 UOS 上开启文件服务器
2. 在开发机上使用下载工具下载文件

如果需要传输一个文件夹呢？其实只需要将文件夹压缩就可以使用上面步骤

1. 在 UOS 上将文件夹压缩成文件
1. 在 UOS 上开启文件服务器
2. 在开发机上使用下载工具下载文件

下面咱一步步来

## 压缩文件夹成文件

这一步在 UOS 上做，可以使用自带的 tar 工具，压缩命令如下

```csharp
 tar -cvf 1.tar lindexi/
```

上面命令就是将 lindexi 文件夹压缩到 1.tar 文件里

详细请看 [Linux tar压缩命令：打包与解打包命令](http://c.biancheng.net/view/788.html )

## 安装 dotnet tool 工具

在 UOS 开启文件服务器可以使用 dotnet-serve 工具，在开始之前需要先初始化一下工具

```csharp
dotnet new tool-manifest
```

上面代码将会在当前工作路径下初始化 dotnet tool 工具

然后使用下面命令安装 dotnet 本机文件服务器工具

```csharp
dotnet tool install dotnet-serve
```

安装完成之后，可以使用下面代码开启文件服务器

```csharp
dotnet tool run dotnet-serve -a 0.0.0.0
```

而在 Windows 端开启文件服务器就没有那么长的命令了，因为 Windows 下是支持全局工具的，也许后续 UOS 也会支持

在 Windows 下第一步是安装工具，注意和 UOS 上安装不同的是添加了 `-g` 表示全局安装，全局安装的工具，可以在任意工作路径下才能使用。而不加上 `-g`的是本地安装，只有在当前当前工作路径下才能使用

```csharp
dotnet tool install -g dotnet-serve
```

接着使用下面代码开启文件服务器

```csharp
dotnet serve -a 0.0.0.0
```

详细请看 [dotnet serve 一句话开启文件服务器 通过 HTTP 将文件共享给其他设备](https://blog.lindexi.com/post/dotnet-serve-%E4%B8%80%E5%8F%A5%E8%AF%9D%E5%BC%80%E5%90%AF%E6%96%87%E4%BB%B6%E6%9C%8D%E5%8A%A1%E5%99%A8-%E9%80%9A%E8%BF%87-HTTP-%E5%B0%86%E6%96%87%E4%BB%B6%E5%85%B1%E4%BA%AB%E7%BB%99%E5%85%B6%E4%BB%96%E8%AE%BE%E5%A4%87.html)

## 下载文件

在 UOS 上是 Linux 系统，可以使用 wget 的工具下载文件，而在 Windows 下可以使用 [dotnetCampus.FileDownloader.Tool](https://github.com/dotnet-campus/dotnetCampus.FileDownloader/) 工具下载文件，这是一个完全在 [GitHub](https://github.com/dotnet-campus/dotnetCampus.FileDownloader/) 开源的项目

安装 dotnet tool 的方法和上面安装文件服务器相同，在 Windows 下使用下面代码作为全局工具安装

```csharp
dotnet tool install -g dotnetCampus.FileDownloader.Tool
```

安装完成之后，可以使用下面命令下载文件

```csharp
DownloadFile -u 下载链接 -o 下载文件保存路径
```

在 UOS 上开启文件服务器，使用下面命令开启

```csharp
dotnet tool run dotnet-serve -a 0.0.0.0
```

再打开一个终端，使用下面命令找到 UOS 的 ip 地址

```csharp
sudo ifconfig
```

假定在 UOS 上开启文件服务器的工作路径下，有刚才压缩好的 1.tar 文件准备传输到 Windows 服务器上

先记下 UOS 的 ip 地址，和执行 dotnet serve 开启的服务器端口，如 36867 端口，此时可以在 Windows 下使用下面命令下载 1.tar 文件

```
downloadfile -u http://172.20.115.72:36867/1.tar -o 1.tar 
```

上面代码的 172.20.115.72 就是 UOS 设备的地址

这样就仅通过工具完成了 UOS 到 Windows 文件的传输，有趣的是，反过来也可以，使用的工具和命令也可以是相同的

更多关于下载工具请看 [https://github.com/dotnet-campus/dotnetCampus.FileDownloader](https://github.com/dotnet-campus/dotnetCampus.FileDownloader)

也许本文的例子还不够让大家看到 dotnet tool 的优势，因为 UOS 作为 Linux 系统，有大量可用的自带的命令行工具，只是存在一个坑就是我对这些工具都是不熟悉的。而 dotnet tool 可以让两边都使用相同的工具和命令

欢迎小伙伴加入 xamarin 国产 UOS 开发群： 810052083 





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。