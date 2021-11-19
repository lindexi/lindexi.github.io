# 迁移 dotnet 6 提示必须将目标平台设置为 Windows 平台

我在迁移一个古老的项目为 .NET 6 框架，但是 VS 提示 error NETSDK1136 如果使用 Windows 窗体或 WPF，或者引用使用 Windows 窗体或 WPF 的项目或包，则必须将目标平台设置为 Windows 平台。但是我不想让此项目绑定到 Windows 平台上，不在 TargetFramework 上修改为 net6.0-windows 框架

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

此提示是在 .NET 的 SDK 的 Microsoft.NET.Sdk.DefaultItems.targets 文件开启的，代码如下

```xml
  <Target Name="_CheckForInvalidWindowsDesktopTargetingConfiguration"
        BeforeTargets="_CheckForInvalidConfigurationAndPlatform"
        Condition="'$(TargetFrameworkIdentifier)' == '.NETCoreApp' and $([MSBuild]::VersionGreaterThanOrEquals($(_TargetFrameworkVersionWithoutV), '5.0')) and ('$(UseWindowsForms)' == 'true' or '$(UseWPF)' == 'true')">
    <NETSdkError Condition="'$(TargetPlatformIdentifier)' != 'Windows'"
                 ResourceName="WindowsDesktopTargetPlatformMustBeWindows" />
  </Target>
```

或者定义在 Microsoft.NET.Sdk.DefaultItems.Shared.targets 代码

```xml
  <Target Name="_CheckForTransitiveWindowsDesktopDependencies"
          AfterTargets="ResolvePackageAssets"
          Condition="'$(TargetFrameworkIdentifier)' == '.NETCoreApp' and $([MSBuild]::VersionGreaterThanOrEquals($(_TargetFrameworkVersionWithoutV), '5.0')) and '$(TargetPlatformIdentifier)' != 'Windows' and '@(TransitiveFrameworkReference)' != ''">
    <ItemGroup>
      <_WindowsDesktopTransitiveFrameworkReference Include="@(TransitiveFrameworkReference)"
                                                   Condition="'%(Identity)' == 'Microsoft.WindowsDesktop.App' Or
                                                              '%(Identity)' == 'Microsoft.WindowsDesktop.App.WPF' Or
                                                              '%(Identity)' == 'Microsoft.WindowsDesktop.App.WindowsForms'" />
    </ItemGroup>
    <NetSdkError Condition="'@(_WindowsDesktopTransitiveFrameworkReference)' != ''"
                 ResourceName="WindowsDesktopTargetPlatformMustBeWindows" />
  </Target>
```

也就是说在 .NET 5.0 和以上的版本，判断到有使用 Windows Forms 或 WPF 就提示需要加上 Windows 平台

这里的判断是包括项目本身和项目所依赖的所有项目和库，只要有一个用到了，将会提示需要加上 Windows 平台

在设置加上 Windows 平台，就自然引用了 Windows Forms 或 WPF 的负载。而我的项目不期望绑定到 Windows 上，就需要去寻找是哪些依赖的项目或库使用到了 Windows Forms 或 WPF 项目

这里必须说明的是 WPF 和 Windows Forms 在啥都不做的情况下，包括 MONO 等都不使用的情况下，是可以有限的支持在 Linux 和 MAC 环境下运行的。什么是有限的支持？只要不碰 Windows 相关平台的逻辑，包括 PInvoke 调用等，那么将可以很好运行

例如只是用到了 WPF 的一些结构体定义，如 Rect 和 Size 等。或者用到了枚举定义，或者用到某些工具方法等等

而我当前的项目刚好就是存在某些依赖库，引用了 WPF 只是为了用到 Size 等定义而已。为了干掉依赖，我换成了 [https://github.com/dotnet-campus/dotnetCampus.WPFType](https://github.com/dotnet-campus/dotnetCampus.WPFType ) 开源库代替引用 WPF 项目。这个开源库拷贝了 WPF 的一些类型定义，基于 MIT 协议

但是在逐步干掉所有的依赖，依然提示如下

```
C:\Program Files\dotnet\sdk\6.0.100\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.DefaultItems.Shared.targets(250,5): error NETSDK1136: 如果使用 Windows 窗体或 WPF，或者引用使用 Windows 窗体或 WPF 的项目或包，则必须将目标平台设置 为 Windows (通常通过在 TargetFramework 属性中添加 "-windows")。
```

实际上原因是在 obj 文件夹里面有缓存，只需要删除 obj 文件夹，重新构建即可

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
