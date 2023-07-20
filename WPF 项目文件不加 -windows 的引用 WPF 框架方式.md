# WPF 项目文件不加 -windows 的引用 WPF 框架方式

默认情况下的 WPF 项目都是带 -windows 的 TargetFramework 方式，但有一些项目是不期望加上 -windows 做平台限制的，本文将介绍如何实现不添加 -windows 而引用 WPF 框架

<!--more-->
<!-- CreateTime:2023/7/17 19:46:06 -->

<!-- 博客 -->
<!-- 发布 -->

先看一个标准的 WPF 项目的 csproj 项目文件内容

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net7.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

</Project>
```

以上代码的核心在于设置 TargetFramework 为 `net7.0-windows` 的同时设置 `UseWPF` 属性。在此设置之下，项目本身就限定了采用 Windows 平台

对于一些特殊的项目来说，也许只是在某些模块下期望引用 WPF 的某些类型，而不想自己的项目限定平台。这时候可以去掉 `-windows` 换成 FrameworkReference 的方式，如以下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <FrameworkReference Include="Microsoft.WindowsDesktop.App" />
    <FrameworkReference Include="Microsoft.WindowsDesktop.App.WPF" />
  </ItemGroup>

</Project>
```

通过 `<FrameworkReference Include="Microsoft.WindowsDesktop.App.WPF" />` 即可设置对 WPF 程序集的引用，也就是仅仅只是将 WPF 的程序集取出来当成引用，而不是加上 WPF 的负载

通过此方式即可不需要设置 TargetFramework 为 `net7.0-windows` 和设置 `UseWPF` 属性

以上方法对于 WinForms 项目来说也是一样的，如果也需要加上 WinForms 程序集的引用，可以添加 `<FrameworkReference Include="Microsoft.WindowsDesktop.App.WindowsForms" />` 如代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <FrameworkReference Include="Microsoft.WindowsDesktop.App" />
    <FrameworkReference Include="Microsoft.WindowsDesktop.App.WPF" />
    <FrameworkReference Include="Microsoft.WindowsDesktop.App.WindowsForms" />
  </ItemGroup>

</Project>
```

以上方法对 .NET 7 或 .NET 6 都生效

