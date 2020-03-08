# git镜像仓库

有时候我们会把一些仓库放到本地，当他更新的时候，可以使用简单命名更新他。

不是所有时间我们都有网，所以把远程的仓库作为镜像，可以方便我们查看

普通的git clone不能下载所有分支，想要简单的git clone所有分支，可以用镜像方法

<!--more-->
<!-- CreateTime:2019/9/2 12:57:37 -->


<div id="toc"></div>

做一个镜像仓库很简单。

进入一个放仓库的文件夹，然后复制他的远程Url

```csharp
git clone --mirror Url

我自己的镜像，假如要把我的https://github.com/lindexi/UWP.git 做镜像到本地
git clone --mirror https://github.com/lindexi/UWP.git
```

这样就有本地镜像，我们需要更新他的时候用

```csharp
git remote update

```
这样我们还要手动更新，那么有没自动更新方法，其实我们还可以用gogs

gogs的下载 https://github.com/gogits/gogs 下载完在Windows下打开

打开需要用命令行

```csharp
gogos.exe web

```

绑定了端口就好

需要使用gogs镜像，添加外部

![](http://image.acmx.xyz/48607e54-9b56-4d1b-a11f-ff44b53046c4201612693440.jpg)

输入地址

![](http://image.acmx.xyz/48607e54-9b56-4d1b-a11f-ff44b53046c4201612693522.jpg)

然后设置他24小时更新

默认就是24小时

我最近在用gogs同步我的本地项目，然后用坚果云同步

gogs 可以我传上csdn，下载：http://download.csdn.net/detail/lindexi_gd/9738926

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
