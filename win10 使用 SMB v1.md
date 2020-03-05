# win10 使用 SMB v1

如果使用网络链接出现`共享需要过时的 SMB1 协议`无法创建映射，那么请看本文

<!--more-->
<!-- CreateTime:2019/8/14 8:55:55 -->

<!-- csdn -->

我在输入网络驱动出现下面的错误

```csharp
因为文件共享不安全，所以你不能连接到文件共享。

共享需要过时的 SMB1 协议，而此协议是不安全的，可能会使你的系统遭受攻击。你的系统需要 SMB2 或更高版本。
```

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201818172624.jpg)

原因是 SMB1 是不安全的，所以微软在 win10 系统就不给使用，如果需要使用，需要使用管理员打开 Powershell 输入下面代码

```csharp
Enable-WindowsOptionalFeature -Online -FeatureName smb1protocol
```

输入之后需要重新启动，如果还是无法使用，请输入下面代码

```csharp
Get-WindowsFeature FS-SMB1
```


参见：https://support.microsoft.com/en-us/help/2696547/how-to-enable-and-disable-smbv1-smbv2-and-smbv3-in-windows-and-windows-server

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。