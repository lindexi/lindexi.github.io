# 如何移动 nuget 缓存文件夹

本文告诉大家如何移动 nuget 缓存文件夹。

因为 nuget 文件夹一般比较大，现在我的 nuget 文件夹有 10 G，默认的 nuget 文件夹是在C盘，所以需要移动他。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->

<!-- 标签：nuget -->
<div id="toc"></div>

可以使用下面的代码查看 nuget 全局缓存文件所在的文件夹

```csharp
nuget locals all -list
```

可以看到下面的输出

```csharp
http-cache: C:\Users\user\AppData\Local\NuGet\v3-cache   #NuGet 3.x+ cache
packages-cache: C:\Users\user\AppData\Local\NuGet\Cache  #NuGet 2.x cache
global-packages: C:\Users\user\.nuget\packages\          #Global packages folder
temp: C:\Users\user\AppData\Local\Temp\NuGetScratch      #Temp folder
```

这样可以看到，所在的全局缓存文件夹是放在 C 盘，那么我提供两个方法可以修改

## 修改链接

可以使用管理员权限运行 PowerShell 来进行文件夹链接，首先复制 nuget 的 package 文件夹到 另外的地方，我移动到`D:\lindexi\packages`，所以就可以使用下面代码把 nuget 文件夹移动到另一个文件夹


```csharp
mklink /d C:\Users\lindexi\.nuget\packages D:\lindexi\packages
```

在使用这个代码之前，需要删除 `C:\Users\lindexi\.nuget\packages` 请把这个字符串修改为自己的 nuget 文件夹

## 配置

除了上面的方法，还可以通过修改配置，修改全局文件夹

打开 %AppData%\\NuGet\\NuGet.Config ，在这个文件夹添加下面代码

```csharp
<configuration>
  <config>
     <add key="globalPackagesFolder" value="D:\lindexi\packages" />
  </config>
</configuration>
```

请把移动的nuget 文件夹修改为你自己的文件夹

```csharp
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
    <add key="cnblog" value="https://nuget.cnblogs.com/v3/index.json" />
  </packageSources>
  <packageRestore>
    <add key="enabled" value="True" />
    <add key="automatic" value="True" />
  </packageRestore>
  <bindingRedirects>
    <add key="skip" value="False" />
  </bindingRedirects>
  <packageManagement>
    <add key="format" value="0" />
    <add key="disabled" value="False" />
  </packageManagement>
  <disabledPackageSources>
    <add key="Microsoft Visual Studio Offline Packages" value="true" />
  </disabledPackageSources>
  <config>
     <add key="globalPackagesFolder" value="D:\Users\linde\.nuget\packages" />
  </config>
</configuration>
```



## NuGet Cache

### Mac

 - ~/.local/share/NuGet/Cache
 - ~/.nuget/packages

### Windows

 - %LocalAppData%\\NuGet\\Cache
 - %UserProfile%\\.nuget\\packages

### Linux

~/.local/share/NuGet/Cache
~/.nuget/packages

## NuGet Configuration

Mac ~/.config/NuGet/NuGet.Config

Windows %AppData%\\NuGet\\NuGet.Config

Linux ~/.config/NuGet/NuGet.Config

参见：[NuGet File Locations - Matt Ward](http://lastexitcode.com/projects/NuGet/FileLocations/ )

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201822814119.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。