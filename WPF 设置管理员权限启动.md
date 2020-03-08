# WPF 设置管理员权限启动

在 dotnet 程序，可以通过清单文件设置管理员权限启动


<!--more-->
<!-- CreateTime:2020/2/1 17:28:47 -->

<!-- 发布 -->

通过下面代码可以判断当前的程序是管理员权限运行

```csharp
            var identity = WindowsIdentity.GetCurrent();
            var principal = new WindowsPrincipal(identity);
            if (principal.IsInRole(WindowsBuiltInRole.Administrator))
            {
                // 当前正在以管理员权限运行。
            }
```

而设置软件启动权限是管理员权限可以添加清单文件，右击添加 App.manifest 文件，此时要求在 csproj 设置` <ApplicationManifest>App.manifest</ApplicationManifest>` 才可以

```
  <PropertyGroup>
      <ApplicationManifest>App.manifest</ApplicationManifest>
  </PropertyGroup>
```

在 App.manifest 文件将 requestedPrivileges 替换下面代码

```
  <trustInfo xmlns="urn:schemas-microsoft-com:asm.v2">
    <security>
      <requestedPrivileges xmlns="urn:schemas-microsoft-com:asm.v3">
        <!-- UAC 清单选项
             如果想要更改 Windows 用户帐户控制级别，请使用
             以下节点之一替换 requestedExecutionLevel 节点。n
        <requestedExecutionLevel  level="asInvoker" uiAccess="false" />
        <requestedExecutionLevel  level="requireAdministrator" uiAccess="false" />
        <requestedExecutionLevel  level="highestAvailable" uiAccess="false" />

            指定 requestedExecutionLevel 元素将禁用文件和注册表虚拟化。
            如果你的应用程序需要此虚拟化来实现向后兼容性，则删除此
            元素。
        -->
        <requestedExecutionLevel level="requireAdministrator" uiAccess="false" />
      </requestedPrivileges>
    </security>
  </trustInfo>
```

如果需要在管理员权限使用降低权限运行请看 [dotnet 判断程序当前使用管理员运行降低权使用普通权限运行](https://blog.lindexi.com/post/dotnet-%E5%88%A4%E6%96%AD%E7%A8%8B%E5%BA%8F%E5%BD%93%E5%89%8D%E4%BD%BF%E7%94%A8%E7%AE%A1%E7%90%86%E5%91%98%E8%BF%90%E8%A1%8C%E9%99%8D%E4%BD%8E%E6%9D%83%E4%BD%BF%E7%94%A8%E6%99%AE%E9%80%9A%E6%9D%83%E9%99%90%E8%BF%90%E8%A1%8C.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
