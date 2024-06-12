本文和大家介绍一个 Hack 的方式，通过此方式可实现访问 UNO 框架里面的 internal 不公开成员，调用 UNO 框架里面的不公开的 API 方法和属性，访问 UNO 里面不公开的类型

<!--more-->


<!-- CreateTime:2024/06/12 07:09:12 -->

<!-- 发布 -->
<!-- 博客 -->

核心原理是基于 UNO 框架里面的 InternalsVisibleToAttribute 程序集特性，指定给到 SamplesApp 等程序集可见。因此只需要新建一个程序集，设置 AssemblyName 为 SamplesApp 即可

以下是我新建的名为 UnoHacker 的项目，此项目和所有的代码都可以在本文末尾找到下载的方法

新建的 UnoHacker 项目选定使用的是 net8.0 的框架，由于本文提供的方式强依赖于 UNO 框架的实现，本文写于 2024.06 如果你阅读本文距离本文编写的时间太长，可能本文将包含不适用于你当前使用的 UNO 框架的知识

本文面向的是 5.2.161 的 UNO 框架版本，不代表后续 UNO 版本也能适用，推荐大家按照本文提供的方式自己进行测试

编辑 UnoHacker 项目的 csproj 项目文件，先配置 AssemblyName 属性，用于指定程序集名，如以下代码

```xml
  <!-- 程序集使用特殊名称，这样才能访问到 internal API。 -->
  <PropertyGroup>
    <AssemblyName>SamplesApp</AssemblyName>
  </PropertyGroup>
```

接下来添加对 Uno.WinUI 的引用，添加此引用只是为了拿到实际的 UNO 引用程序集而已，而不是真的需要引用此包。这也就是为什么以下代码需要添加不使用任何内容的 `ExcludeAssets="all"` 代码。以下代码还添加了 GeneratePathProperty 属性配置，通过此属性配置可以用于拿到对应的包在缓存文件夹的路径，用于引用包里面的内容

```xml
  <!-- 仅为获取到 NuGet 目录，不使用任何内容（包括 compile;runtime;build 等） -->
  <ItemGroup>
    <PackageReference Include="Uno.WinUI" GeneratePathProperty="true" PrivateAssets="all" ExcludeAssets="all" />
  </ItemGroup>
```

由于默认使用了中央包管理，以上代码引用可以不添加版本号。添加以上代码之后，即可确保本地存在 NuGet 包，且通过 GeneratePathProperty 属性配置，可以通过 `$(PKGUno_WinUI)` 属性拿到 Uno.WinUI 这个包的本地路径

在我设备上拿到的 `$(PKGUno_WinUI)` 属性的内容如下

```
C:\Users\lindexi\.nuget\packages\uno.winui\5.2.139
```

通过此即可拼接路径，拿到 NuGet 包里面的文件，如以下代码使用了 UNO 真正在 Skia 平台下的发布文件

```xml
  <!-- 引用 net8.0-desktop 真正生效的程序集，而不是专供引用的程序集。 -->
  <ItemGroup>
    <Reference Include="Uno" Private="False">
      <HintPath>$(PKGUno_WinUI)\uno-runtime\net8.0\skia\Uno.dll</HintPath>
    </Reference>
    <Reference Include="Uno.UI" Private="False">
      <HintPath>$(PKGUno_WinUI)\uno-runtime\net8.0\skia\Uno.UI.dll</HintPath>
    </Reference>
    <Reference Include="Uno.UI.Composition" Private="False">
      <HintPath>$(PKGUno_WinUI)\uno-runtime\net8.0\skia\Uno.UI.Composition.dll</HintPath>
    </Reference>
  </ItemGroup>
```

这里需要额外说明的是 UNO 让大家写代码引用的程序集，和发布后最终输出的 DLL 程序集不是相同的文件。让大家编写代码使用的是 NuGet 包里面 Lib 文件夹下的，而实际发布输出的是 DLL 是在 uno-runtime 下的。通过使用不同的 DLL 即可让 UNO 更好的支持多个不同的平台，对于不同的平台可使用不同的 DLL 输出

完成以上代码之后的 csproj 项目文件的代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <OutputType>Library</OutputType>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <DefineConstants>$(DefineConstants);HAS_UNO</DefineConstants>

    <!--
      UnoFeatures let's you quickly add and manage implicit package references based on the features you want to use.
      https://aka.platform.uno/singleproject-features
    -->
    <!--
    <UnoFeatures></UnoFeatures>
    -->
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="SkiaSharp" />
  </ItemGroup>

  <!-- ================ !!! UNO HACKER !!! ================ -->

  <!-- 程序集使用特殊名称，这样才能访问到 internal API。 -->
  <PropertyGroup>
    <AssemblyName>SamplesApp</AssemblyName>
  </PropertyGroup>

  <!-- 仅为获取到 NuGet 目录，不使用任何内容（包括 compile;runtime;build 等） -->
  <ItemGroup>
    <PackageReference Include="Uno.WinUI" GeneratePathProperty="true" PrivateAssets="all" ExcludeAssets="all" />
  </ItemGroup>

  <!-- 引用 net8.0-desktop 真正生效的程序集，而不是专供引用的程序集。 -->
  <ItemGroup>
    <Reference Include="Uno" Private="False">
      <HintPath>$(PKGUno_WinUI)\uno-runtime\net8.0\skia\Uno.dll</HintPath>
    </Reference>
    <Reference Include="Uno.UI" Private="False">
      <HintPath>$(PKGUno_WinUI)\uno-runtime\net8.0\skia\Uno.UI.dll</HintPath>
    </Reference>
    <Reference Include="Uno.UI.Composition" Private="False">
      <HintPath>$(PKGUno_WinUI)\uno-runtime\net8.0\skia\Uno.UI.Composition.dll</HintPath>
    </Reference>
  </ItemGroup>

  <!-- ================ !!! UNO HACKER !!! ================ -->
</Project>
```

尝试编写代码测试访问 UNO 里面的不公开的成员

```csharp
using Windows.UI.ViewManagement;
using Microsoft.UI.Windowing;
#if HAS_UNO
using Windows.UI;
using Uno.UI.Xaml.Core;
#endif

namespace UnoHacker;

public static class ApplicationViewExtension
{
#if HAS_UNO
    public static ApplicationView GetApplicationView(this AppWindow appWindow) =>
        ApplicationView.GetForWindowId(appWindow.Id);
#endif
}
```

可以看到代码编写非常方便，且避免使用反射，其性能更高

通过此方式即可使用到一些 UNO 里面不公开的成员，从而实现一些特定的需求。但必须说明的是 UNO 不对不公开的 API 进行稳定性承诺，大家使用的时候需要进行足够的测试

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/5a2adba87164ab5c2de480cb4f04a8e28bb28bce/UnoDemo/UnoSkiaWeelelqairjiwarfekemGahinabai) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/5a2adba87164ab5c2de480cb4f04a8e28bb28bce/UnoDemo/UnoSkiaWeelelqairjiwarfekemGahinabai) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 5a2adba87164ab5c2de480cb4f04a8e28bb28bce
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 5a2adba87164ab5c2de480cb4f04a8e28bb28bce
```

获取代码之后，进入 UnoDemo/UnoSkiaWeelelqairjiwarfekemGahinabai 文件夹，即可获取到源代码

由 new bing 翻译的英文版本：[Accessing Internal Members in the UNO Framework](https://blog.lindexi.com/post/Accessing-Internal-Members-in-the-UNO-Framework.html )
<!-- [Accessing Internal Members in the UNO Framework - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18243187 ) -->
