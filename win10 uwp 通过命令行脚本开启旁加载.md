# win10 uwp 通过命令行脚本开启旁加载

在 UWP 旁加载安装，需要用户的设备上开启旁加载功能，这个功能需要点击设置，点击更新，找到开发者选项，点击开启旁加载。这对用户来说要安装一个应用需要点这么多步骤，基本上很少用户能成功。本文告诉大家如何通过命令行或通过脚本的方式协助用户开启旁加载的功能

<!--more-->
<!-- CreateTime:2019/11/25 15:45:34 -->

<!-- csdn -->

在 win10 的旁加载其实就是写入注册表，允许所有应用信任安装，所以只需要通过命令行写入注册表就可以

通过调用 reg 写入注册表，可以使用下面命令

```csharp
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\AppModelUnlock" /t REG_DWORD /f /v "AllowAllTrustedApps" /d "1"
```

也就是通过在管理员权限命令行输入命令就可以在注册表修改

如果需要开启开发者模式，那么可以使用下面命令

```csharp
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\AppModelUnlock" /t REG_DWORD /f /v "AllowDevelopmentWithoutDevLicense" /d "1"
```

如果使用 PowerShell 脚本，代码有点多

```csharp
$registryPath = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\AppModelUnlock" 
 
$Name1 = "AllowAllTrustedApps" 
$value1 = "1" 
New-ItemProperty -Path $registryPath -Name $name1 -Value $value1 -PropertyType DWORD -Force 
```

上面是开启旁加载的脚本，如果还要开启开发者模式，那么在上面的代码添加下面代码

```csharp
$Name2 = "AllowDevelopmentWithoutDevLicense" 
$value2 = "0" 
 
New-ItemProperty -Path $registryPath -Name $name2 -Value $value2 -PropertyType DWORD -Force
```

通过命令行开启旁加载之后就可以安装旁加载应用

开启旁加载可以使用 [通过win32安装UWP应用](https://blog.lindexi.com/post/%E5%8A%A0%E5%BC%BA%E7%89%88%E5%9C%A8%E5%9B%BD%E5%86%85%E5%88%86%E5%8F%91-UWP-%E5%BA%94%E7%94%A8%E6%AD%A3%E7%A1%AE%E6%96%B9%E5%BC%8F-%E9%80%9A%E8%BF%87win32%E5%AE%89%E8%A3%85UWP%E5%BA%94%E7%94%A8.html ) 博客说的将 UWP 应用通过win32安装包给用户，解决应用商店下载

[Powershell script to Enable Sideloading for Windows 10 v1.0](https://gallery.technet.microsoft.com/scriptcenter/Powershell-script-to-ccb46131)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
