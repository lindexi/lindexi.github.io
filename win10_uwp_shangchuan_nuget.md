# win10 uwp 上传Nuget 让别人用我们的库

## Nuget


我们的开发经常使用别人的dll，那么我们需要每次都从网上下载，然后复制到我们的项目，
而不知道我们的dll是否安全？

当我们的库更新的时候，我们又需要从网上搜索，这样不好，于是我们就用Nuget，Nuget可以
让我们把我们做出来的东西发在网上让别人下载，需要搜索名称就可以下载，然后更新会通知

本文主要讲我们如何制作一个库上传Nuget，让别人可以使用，做法很简单，先下载软件后制作，上传。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>


## 下载

首先下载

nuget.exe https://dist.nuget.org/win-x86-commandline/latest/nuget.exe

如果没法下载和我说，我发给你

然后下载一个工具https://docs.nuget.org/Create/using-a-gui-to-build-packages，下载后运行

我们把Nuget.exe放在Path，当然不知道Path,就放在我们要打包的工程文件夹


## 打包Nuget

我们用MSBuild命令进入项目文件夹，如果找不到MsBuild就用cmd

项目文件夹就是我们要打包项目*.csproj所在文件夹

我的工程文件smms，打开smms.csproj所在文件夹A:\smms\smms

进入文件夹命令

首先进入盘，我是在A盘，

```csharp
a:
```

然后进入文件夹

```csharp
cd smms/smms
```

![这里写图片描述](http://img.blog.csdn.net/20160705153953828)

我们打包

首先

`nuget spec`

![这里写图片描述](http://img.blog.csdn.net/20160705154308176)

`smms.nuspec` 这个可以用文本打开，里面会自动替换

里面有

```xml
<?xml version="1.0"?>
<package >
  <metadata>
    <id>$id$</id>
    <version>$version$</version>
    <title>$title$</title>
    <authors>$author$</authors>
    <owners>$author$</owners>
    <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
    <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
    <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>$description$</description>
    <releaseNotes>Summary of changes made in this release of the package.</releaseNotes>
    <copyright>Copyright 2016</copyright>
    <tags>Tag1 Tag2</tags>
  </metadata>
</package>
```

- id 
  包的ID：必须的唯一的ID，格式和C#命名空间规范相同，在你发布包的时候会去验证唯一性。

- version 
  版本号：必须的三段式的版本号，注意每次发布必须大于上一次的版本号，否则将会被nuget驳回。

- title 
  标题：非必需的，通常你可以让它和ID保持一致，但是这不是强制的。

- authors
  作者(s)：必须的项目，以逗号分隔作者列表。

- owners 
  拥有者：你可以随便写，但是在发布的时候会被你的nuget帐户名替代。

- 最低客户端版本：描述这个包限制的最低nuget客户端版本。

- iconUrl
  一个`32*32`像素的.png文件地址，作为最终在nuget中显示的图标

- 描述、标签、许可地址、项目地址

- Dependencies
  我们发的依赖其他程序，那就写他，具体怎么我还不知，我就放空，不管，好像写的要在NuGet服务器上能找到，否则无法完成自动引用添加

我们可以在vs，Properties打开，写上我们名字和说明

![这里写图片描述](http://img.blog.csdn.net/20160705154334553)

![这里写图片描述](http://img.blog.csdn.net/20160705154345051)

把全部写后生成

其中他会把`$$`代为AssemblyInfo.cs 

作者代为AssemblyCompany

id代为Assembly名

version代为AssemblyVersion

description代为AssemblyDescription

做完我们文本打开 `*.nuspec`

改releaseNotes、tags

如果没有修改，我们打包 `nuget pack *.csproj`

![这里写图片描述](http://img.blog.csdn.net/20160705154419364)

```csharp
问题: 删除示例 nuspec 值。
说明: Tags 的值“Tag1 Tag2”是示例值，应将其删除。
解决方案: 请替换为适当的值或删除它，然后重新生成程序包。

问题: 删除示例 nuspec 值。
说明: ReleaseNotes 的值“Summary of changes made in this release of the package.”是示例值，应将其删除。
解决方案: 请替换为适当的值或删除它，然后重新生成程序包。
```

把我们信息写后打包

```csharp
nuget pack smms.csproj
```

![这里写图片描述](http://img.blog.csdn.net/20160705154443317)

我们就把我们项目打包，接着我们看到文件夹有`*.nupkg`

修改项目地址

![这里写图片描述](http://img.blog.csdn.net/20160705154503646)


## 上传

首先有一个微软账号，登录 [https://www.nuget.org](https://www.nuget.org)

点击自己 [https://www.nuget.org/account](https://www.nuget.org/account)

![这里写图片描述](http://img.blog.csdn.net/20160705154531195)

复制，这个key是我的，你的应该和我不同

![这里写图片描述](http://img.blog.csdn.net/20160705154957007)

刚才复制的

![这里写图片描述](http://img.blog.csdn.net/20160705154606068)

![这里写图片描述](http://img.blog.csdn.net/20160705154624787)

发布

我们可以在[https://www.nuget.org/account/Packages](https://www.nuget.org/account/Packages)

我们上传包，如果还要上传，我们的version要比之前大

我们在

![这里写图片描述](http://img.blog.csdn.net/20160705155015936)

搜索不到，不过我们还是上传了

安装

`Install-Package ID`

![这里写图片描述](http://img.blog.csdn.net/20160705155205298)

安装完搜索就可以搜索到

![这里写图片描述](http://img.blog.csdn.net/20160705155225430)

参见：http://www.cnblogs.com/xiaoyaojian/p/4199735.html

## 命令行使用Nuget

参见：https://docs.nuget.org/consume/command-line-reference

[[.Net] 手把手带你将自己打造的类库丢到 NuGet 上 - 反骨仔（二五仔） - 博客园](http://www.cnblogs.com/liqingwen/p/5859236.html)

还可以使用比较快的镜像 [NuGet镜像上线试运行 - 博客园团队 - 博客园](http://www.cnblogs.com/cmt/p/nuget-mirror.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

