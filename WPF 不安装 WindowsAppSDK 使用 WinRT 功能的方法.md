# WPF 不安装 WindowsAppSDK 使用 WinRT 功能的方法

安装 Microsoft.WindowsAppSDK 库会限制应用程序只能分发 windows 10 应用，如果自己的应用程序依然需要兼容 Win7 等旧系统，那直接采用安装 WindowsAppSDK 方法将会丧失兼容旧系统能力。本文和大家介绍无需安装 Microsoft.WindowsAppSDK 即可使用 WinRT 功能的方法，此方法可以让应用程序继续兼容旧系统，可以在应用程序内判断系统版本之后自行决定调用 WinRT 功能

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

本文的方法使用起来特别简单，只需要让自己的项目引用 `Microsoft.Windows.SDK.NET.dll` 和 `WinRT.Runtime.dll` 即可

例如将 `Microsoft.Windows.SDK.NET.dll` 和 `WinRT.Runtime.dll` 放在项目文件夹之外的 bin 文件夹里面，修改自己的 csproj 项目文件让其引用。或者是在 Visual Studio 里面右击项目添加引用都可以。修改之后的 csproj 项目文件大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
    <PlatformTarget>x86</PlatformTarget>
  </PropertyGroup>

  <ItemGroup>
    <Reference Include="Microsoft.Windows.SDK.NET">
      <HintPath>..\bin\Microsoft.Windows.SDK.NET.dll</HintPath>
    </Reference>
    <Reference Include="WinRT.Runtime">
      <HintPath>..\bin\WinRT.Runtime.dll</HintPath>
    </Reference>
  </ItemGroup>
  
</Project>
```

不同的开发者将 `Microsoft.Windows.SDK.NET.dll` 和 `WinRT.Runtime.dll` 放置的地方不同，这将会导致 csproj 里的 HintPath 的路径有些差别

那接下来的问题就是如哪里找到正确的 `Microsoft.Windows.SDK.NET.dll` 和 `WinRT.Runtime.dll` 文件？这两个文件是否区分 x86 或 x64 版本？

这两个文件是从 Microsoft.WindowsAppSDK 库里面找出来的，因此咱可以通过安装 Microsoft.WindowsAppSDK 库来找到这两个文件。方法就是先创建一个新项目，在这个新项目上安装上 Microsoft.WindowsAppSDK 库，接着构建这个新项目。在新项目的构建输出，如 bin\Debug 文件夹下找到 `Microsoft.Windows.SDK.NET.dll` 和 `WinRT.Runtime.dll` 文件，取出来即可

这个专门用来获取 `Microsoft.Windows.SDK.NET.dll` 和 `WinRT.Runtime.dll` 文件的新项目的 csproj 项目文件可以是大概如下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows10.0.19041</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
    <PlatformTarget>x86</PlatformTarget>
    <RuntimeIdentifiers>win10-x86;win10-x64</RuntimeIdentifiers>
    <TargetPlatformMinVersion>10.0.17763.0</TargetPlatformMinVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.3.230331000" />
  </ItemGroup>
  
</Project>
```

以上代码的 Microsoft.WindowsAppSDK 的版本还请大家根据具体使用的时间自行更新版本

这两个文件是不区分 x86 和 x64 的。同样也不区分 Debug 和 Release 版本

至于为什么不区分，那是因为从原理上来说这两个文件其实只是一层 COM 封装而已。对 WinRT 的调用的机制原理就是通过 COM 调用技术来调用到 WinRT 组件。但为什么很少有开发者自己定义 COM 封装去调用 WinRT 呢？其中一个原因是 WinRT 里面用到许多有趣的特性，例如异步等等，导致封装的代码不能和 Win32 那么清真

<!-- 微软封装的 `WinRT.Runtime.dll` 就是对 WinRT 的 COM 的底层封装。底层封装用起来大家都不会很开森，于是微软就继续在 `WinRT.Runtime.dll` 基础上封装了  -->

微软封装的 `Microsoft.Windows.SDK.NET.dll` 和 `WinRT.Runtime.dll` 文件就是对 WinRT 的 COM 的封装，从而让大家可以更加开森和清真的使用到 WinRT 功能。由于封装 COM 的代码也是基础的 C# 代码，这也就无视 x86 和 x64 的差别

再有一个问题是这两个文件是免费商用许可的？答案自然是肯定的。这两个文件来源于 [https://github.com/microsoft/WindowsAppSDK](https://github.com/microsoft/WindowsAppSDK) 开源仓库，这两个文件的代码都是完全开源的，而且还是基于最友好的 MIT 协议进行开源

这就意味着如果大家想不开要改代码也是没有限制门槛，可以放心在自己的商业项目里使用，也可以魔改之后使用