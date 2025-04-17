
# Ubuntu 通过本机代理修复 NuGet 还原 error NU1301 失败

在国内垃圾的网络环境下，我在虚拟机里面安装了 Ubuntu 系统，准备用来测试 MAUI 在 Linux 上的行为，然而使用 dotnet restore 构建时，提示 NU1301 失败。我通过配置环境变量的方式，让 nuget 还原使用上我物理机的网络代理，成功将 NuGet 包下载

<!--more-->


<!-- CreateTime:2022/8/16 21:13:19 -->

<!-- 发布 -->

我在 Ubuntu 系统上，刚开始使用了 dotnet build 命令开始构建，提示的错误有些诡异，似乎是 NuGet 的 Restore 失败。于是我换用 dotnet restore 命令，此时了解到失败的原因是无法连接上 nuget.org 下载内容

```
error NU1301: Unable to load the service index for source https://api.nuget.org/v3/index.json.
```

我的物理机器上有网络代理，可以提供更快的方式访问网络。我的网络代理有提供局域网的 http 代理，于是我在命令行输入以下命令，用来开启网络代理

```
export http_proxy=http://192.168.0.1:10113
```

以上的 192.168.0.1 是我的物理机相对虚拟机的 ip 地址，我的物理机上面有多个不同的地址，我是一个个测试找到的。端口 10113 是物理机的网络代理开放的局域网端口

输入以上的命令设置代理，再执行 dotnet restore 命令，看到了还原是走网络代理，等待一会，成功还原





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。