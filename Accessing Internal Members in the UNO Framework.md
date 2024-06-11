# Accessing Internal Members in the UNO Framework

This article introduces a hack that allows access to internal, non-public members of the UNO framework. This includes calling non-public API methods and properties, and accessing non-public types within UNO.

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

The core principle is based on the `InternalsVisibleToAttribute` assembly attribute in the UNO framework, which is made visible to assemblies such as `SamplesApp`. Therefore, all you need to do is create a new assembly and set the `AssemblyName` to `SamplesApp`.

Below is a new project I created called `UnoHacker`. You can find all the code for this project at the end of this article.

The `UnoHacker` project I created uses the `net8.0` framework. Since the method provided in this article strongly depends on the implementation of the UNO framework, this article was written in June 2024. If you read this article a long time after it was written, it may contain knowledge that is not applicable to the UNO framework you are currently using.

This article is aimed at version `5.2.161` of the UNO framework. It does not represent that subsequent UNO versions will also be applicable. It is recommended that you test it yourself according to the method provided in this article.

Edit the `csproj` project file of the `UnoHacker` project, first configure the `AssemblyName` property to specify the assembly name, as shown in the following code:

```xml
  <!-- The assembly uses a special name so that it can access the internal API. -->
  <PropertyGroup>
    <AssemblyName>SamplesApp</AssemblyName>
  </PropertyGroup>
```

Next, add a reference to `Uno.WinUI`. Adding this reference is just to get the actual UNO reference assembly, not really needing to reference this package. That's why the following code needs to add `ExcludeAssets="all"` code that does not use any content. The following code also adds the `GeneratePathProperty` attribute configuration, which can be used to get the path of the corresponding package in the cache folder through this attribute configuration, which is used to reference the content in the package.

```xml
  <!-- Only to get the NuGet directory, do not use any content (including compile;runtime;build, etc.) -->
  <ItemGroup>
    <PackageReference Include="Uno.WinUI" GeneratePathProperty="true" PrivateAssets="all" ExcludeAssets="all" />
  </ItemGroup>
```

Since central package management is used by default, the above code reference does not need to add a version number. After adding the above code, you can ensure that the NuGet package exists locally, and through the `GeneratePathProperty` attribute configuration, you can get the local path of the `Uno.WinUI` package through the `$(PKGUno_WinUI)` attribute.

The content of the `$(PKGUno_WinUI)` attribute I got on my device is as follows:

```
C:\Users\lindexi\.nuget\packages\uno.winui\5.2.139
```

Through this, you can splice the path and get the files in the NuGet package. The following code uses the actual UNO under the Skia platform:

```xml
  <!-- Reference the net8.0-desktop assembly that is really effective, not the assembly dedicated to reference. -->
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

What needs to be explained here is that the assembly that UNO allows you to write code references, and the DLL assembly that is finally output after publishing are not the same file. The assembly that allows you to write code is under the Lib folder in the NuGet package, and the actual published output is the DLL is under `uno-runtime`. By using different DLLs, UNO can better support multiple different platforms, and different DLL outputs can be used for different platforms.

The code of the `csproj` project file after completing the above code is roughly as follows:

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

  <!-- The assembly uses a special name so that it can access the internal API. -->
  <PropertyGroup>
    <AssemblyName>SamplesApp</AssemblyName>
  </PropertyGroup>

  <!-- Only to get the NuGet directory, do not use any content (including compile;runtime;build, etc.) -->
  <ItemGroup>
    <PackageReference Include="Uno.WinUI" GeneratePathProperty="true" PrivateAssets="all" ExcludeAssets="all" />
  </ItemGroup>

  <!-- Reference the net8.0-desktop assembly that is really effective, not the assembly dedicated to reference. -->
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

Try to write code to test access to UNO's non-public members:

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

You can see that the code is very convenient to write, and it avoids using reflection, which has higher performance.

Through this method, you can use some non-public members in UNO to achieve some specific requirements. But it must be stated that UNO does not make a stability commitment to non-public APIs. You need to conduct sufficient tests when using them.

The code of this article is placed on [github](https://github.com/lindexi/lindexi_gd/tree/5a2adba87164ab5c2de480cb4f04a8e28bb28bce/UnoDemo/UnoSkiaWeelelqairjiwarfekemGahinabai) and [gitee](https://gitee.com/lindexi/lindexi_gd/tree/5a2adba87164ab5c2de480cb4f04a8e28bb28bce/UnoDemo/UnoSkiaWeelelqairjiwarfekemGahinabai). You can use the following command line to pull the code:

First create an empty folder, then use the cd command to enter this empty folder in the command line, enter the following code in the command line, and you can get the code of this article:

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 5a2adba87164ab5c2de480cb4f04a8e28bb28bce
```

The above uses the gitee source. If gitee cannot be accessed, please replace it with the github source. Please continue to enter the following code in the command line to switch the gitee source to the github source to pull the code:

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 5a2adba87164ab5c2de480cb4f04a8e28bb28bce
```

After obtaining the code, enter the `UnoDemo/UnoSkiaWeelelqairjiwarfekemGahinabai` folder to get the source code.
