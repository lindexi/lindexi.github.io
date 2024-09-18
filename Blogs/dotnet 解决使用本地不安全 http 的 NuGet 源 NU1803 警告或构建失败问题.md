---
title: dotnet 解决使用本地不安全 http 的 NuGet 源 NU1803 警告或构建失败问题
description: 出于安全性考虑，微软在 NuGet 的 6.3 版本开始引入 NU1803 警告，此警告将在遇到使用的 NuGet 源为 http 源时触发。 微软推荐 NuGet 的源应该都是安全的 https 源，甚至在 dotnet 9 预览版本里面将其视为构建错误

<!--more-->

tags: dotnet
category: 
---

<!-- CreateTime:2024/07/20 07:19:28 -->

<!-- 发布 -->
<!-- 博客 -->

在微软的 [NuGet is HTTPS everywhere](https://devblogs.microsoft.com/nuget/https-everywhere/) 文档里面说明了此决策的原因。但同时在许多开发环境中，将会使用到内部或本地的 http 源，比如说公司或团队内部搭建的 nuget 源。尽管使用的是不安全的 http 协议，但对于本地或内部源来说，完全不会因此导致安全性问题

在 2024 的 10 月之前，咱依然可以使用 NoWarn 配置忽略 NU1803 警告，如以下代码

```xml
<NoWarn>$(NoWarn);NU1803</NoWarn>
```

以上代码可以写到你的 csproj 项目文件里面，也可以放在 `Directory.Build.props` 做全局忽略。将 NoWarn 放入到 PropertyGroup 里即可

```xml
  <PropertyGroup>
    ... 忽略其他配置
    <!--
      添加 NoWarn 以移除构建警告
      NU1803: 使用了 http 不安全的 NuGet 源
    -->
    <NoWarn>$(NoWarn);NU1803</NoWarn>
  </PropertyGroup>
```

在此时间之后，微软也许会直接让使用 http 协议的 NuGet 源的项目构建不通过。咱如果确认本地或内部的 NuGet 源安全，在 NuGet 的 6.8 以上版本，可在 NuGet 源里添加 `allowInsecureConnections` 配置，编辑之后的 `NuGet.config` 文件里面配置的包源的代码如下

```xml
<packageSources>
    <add key="http-source" value="http://httpsourcetrusted/" allowInsecureConnections="true" />
</packageSources>
```

以上的 `allowInsecureConnections` 的含义如下

> When false, or not specified, NuGet will emit a warning when the source uses http, rather than https. If you are confident that communication with this source will never be at risk of interception attacks, you can set the value to true to suppress the warning. Supported in NuGet 6.8+.

详细请看 <https://learn.microsoft.com/en-us/nuget/reference/nuget-config-file#packagesources>

以上的 `NuGet.config` 可以放在项目的 sln 所在的文件夹，随着项目走。也可以存放在本机里作为全局配置，本机路径分别如下

- Windows: 
  - 用户级： `%appdata%\NuGet\NuGet.Config`
  - 机器级： ``
- Mac/Linux: 
  - 用户级： `~/.config/NuGet/NuGet.Config` 或 `~/.nuget/NuGet/NuGet.Config`
  - 机器级： `/etc/opt/NuGet/Config` (Linux) 和 `/Library/Application Support` (Mac)

参考文档：

- <https://github.com/NuGet/Home/issues/12013>
- <https://github.com/NuGet/Home/issues/12015>
- <https://learn.microsoft.com/en-us/nuget/reference/nuget-config-file#packagesources>
