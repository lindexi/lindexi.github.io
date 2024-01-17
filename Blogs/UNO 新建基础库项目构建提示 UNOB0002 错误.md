当咱新建了一个 UNO 的基础库或被引用的项目时，可能采用的是默认的基础库或库项目创建方法，被引用的程序集没有带上 WinUI 的黑科技，导致构建提示 UNOB0002: Project XX contains a reference to Uno Platform but does not contain a WinAppSDK compatible target framework. 失败

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

如下面代码，这是默认我新建的项目的代码，以下代码已经添加了对 `windows10.0.19041` 的引用了

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows')) or '$(EnableWindowsTargeting)' == 'true'">$(TargetFrameworks);$(DotNetVersion)-windows10.0.19041</TargetFrameworks>
    <TargetFrameworks>$(TargetFrameworks);$(DotNetVersion);</TargetFrameworks>
    <TargetFrameworks Condition="'$(OverrideTargetFramework)'!=''">$(OverrideTargetFramework)</TargetFrameworks>

    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Uno.WinUI" />
  </ItemGroup>
</Project>
```

然而以上代码依然会在构建时失败

遇到错误 UNOB0002: Project XX contains a reference to Uno Platform but does not contain a WinAppSDK compatible target framework.

根据 UNO 官方的文档的如下描述

> This error code means that a WinAppSDK project is referencing a project in your solution which is not providing a net7.0-windows10.xx TargetFramework.
> This can happen if a project contains only a net7.0 TargetFramework and has a NuGet reference to Uno.WinUI.
> To fix this, it is best to start from a Cross Platform Library project template provided by the Uno Platform visual studio extension, or using dotnet new.

解决方法是通过 VisualStudio 模版新建为 UNO 的 Library 项目，或者替换 csproj 为以下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows')) or '$(EnableWindowsTargeting)' == 'true'">$(TargetFrameworks);net7.0-windows10.0.19041</TargetFrameworks>
    <TargetFrameworks>$(TargetFrameworks);net7.0;net7.0-ios;net7.0-maccatalyst;net7.0-android</TargetFrameworks>
    <!-- Ensures the .xr.xml files are generated in a proper layout folder -->
    <GenerateLibraryLayout>true</GenerateLibraryLayout>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Uno.WinUI" />
  </ItemGroup>
  <Choose>
    <When Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">
      <ItemGroup>
        <PackageReference Include="Microsoft.WindowsAppSDK"  />
        <PackageReference Include="Microsoft.Windows.SDK.BuildTools" />
      </ItemGroup>
      <ItemGroup>
        <!--
				If you encounter this error message:

					error NETSDK1148: A referenced assembly was compiled using a newer version of Microsoft.Windows.SDK.NET.dll.
					Please update to a newer .NET SDK in order to reference this assembly.

				This means that the two packages below must be aligned with the "build" version number of
				the "Microsoft.Windows.SDK.BuildTools" package above, and the "revision" version number
				must be the highest found in https://www.nuget.org/packages/Microsoft.Windows.SDK.NET.Ref.
				-->
        <!-- <FrameworkReference Update="Microsoft.Windows.SDK.NET.Ref" RuntimeFrameworkVersion="10.0.22621.28" />
				<FrameworkReference Update="Microsoft.Windows.SDK.NET.Ref" TargetingPackVersion="10.0.22621.28" /> -->
      </ItemGroup>
    </When>
    <Otherwise>
      <ItemGroup>
        <Content Include="Assets\**" />
        <Page Include="**\*.xaml" Exclude="bin\**\*.xaml;obj\**\*.xaml" />
        <Compile Update="**\*.xaml.cs">
          <DependentUpon>%(Filename)</DependentUpon>
        </Compile>
        <PriResource Include="**\*.resw" />
      </ItemGroup>
    </Otherwise>
  </Choose>
  <ItemGroup>
    <UpToDateCheckInput Include="**\*.xaml" Exclude="bin\**\*.xaml;obj\**\*.xaml" />
  </ItemGroup>
</Project>
```

由于代码比较长，推荐大家还是使用自带的模版创建比较方便




本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/02e2942f9b223eb2dcc66f19441a8f68f2200837/LacebayjeejiBehebilawla) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/02e2942f9b223eb2dcc66f19441a8f68f2200837/LacebayjeejiBehebilawla) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 02e2942f9b223eb2dcc66f19441a8f68f2200837
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 02e2942f9b223eb2dcc66f19441a8f68f2200837
```

获取代码之后，进入 LacebayjeejiBehebilawla 文件夹
