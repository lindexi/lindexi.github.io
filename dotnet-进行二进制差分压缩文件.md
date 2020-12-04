
# dotnet 进行二进制差分压缩文件

我需要对一个文件做二进制差分压缩，我有一个文件的起始点，在之后的每次更改我都记录文件的二进制的差分，这样就可以通过起始点和差分文件计算修改后的文件。通过二进制差分可以用来提高文件保存磁盘读写速度，也可以减少软件自动更新需要的文件大小

<!--more-->


<!-- CreateTime:2019/12/24 9:27:49 -->



在 QQ 和 Chrome 等软件更新就使用这个技术，通过二进制差分方法下载差分文件，然后用差分文件和当前版本计算出新版本，将新版本写入文件。这样做的优势在于每次版本的二进制更改的内容都不多，此时可以减少进行 OTA 下载的文件大小

而在弱网的传输也是，这里的弱网是网络下载速度很慢的环境，需要更新某些文件。如游戏端的资源文件，在游戏端为了提升性能，会将资源文件合并为一个文件，在游戏需要更新时候，如果更新整个文件，下载的内容将会很多，在移动端也许用户使用的是流量下载。因此优化方法是读取这个资源的二进制差分文件，只下载之前二进制文件不存在的二进制内容，然后根据现有的二进制文件计算出新的版本的文件

本文使用的算法请看 [二进制数据差异算法 用于减小OTA内容](https://blog.lindexi.com/post/%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%95%B0%E6%8D%AE%E5%B7%AE%E5%BC%82%E7%AE%97%E6%B3%95-%E7%94%A8%E4%BA%8E%E5%87%8F%E5%B0%8FOTA%E5%86%85%E5%AE%B9.html) 这些算法相关的也没有什么内容

和 BSDiff 这个二进制压缩算法相比，本文使用的方法更适合做 dotnet 的更新程序，测试效果将会更好。但是存在的缺点是如果文件比较大，那么计算出二进制差异文件的时间将会很长。这里说的计算时间指的是服务器端根据两个版本的文件，计算出二进制差异文件。在客户端下载了二进制差异文件，然后通过本地的文件计算出新的版本的文件的速度依然是很快的

本文的算法如果作为 .NET 软件的 OTA 自动更新使用，需要构建的时候开启确定性构建，请看 [Roslyn 的确定性构建 - walterlv](https://blog.walterlv.com/post/deterministic-builds-in-roslyn.html )

可以在 csproj 添加下面代码

```xml
<Project>
 <PropertyGroup>
   <Deterministic>true</Deterministic>
 </PropertyGroup>
</Project>
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/blob/739bb867bd62d9356dc5a3d189e9e1d63daf4a69/LwufxgbaDljqkx/) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。