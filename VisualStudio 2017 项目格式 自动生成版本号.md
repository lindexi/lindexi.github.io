# VisualStudio 2017 项目格式 自动生成版本号

最近我把很多项目都使用了 VisualStudio 2017 新项目格式，在使用的时候发现一些比较好用的功能。

本文告诉大家如何使用 VisualStudio 2017 项目格式自动生成版本号

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->

<!-- csdn -->
<!-- 标签：VisualStudio -->


在看本文之前，我认为大家都不是第一次接触 VisualStudio 2017 项目格式。新的项目格式是比较简单的，但是也有一些设置项是比较复杂。

创建一个 UWP 使用 VisualStudio 2017 项目格式请看[将 WPF、UWP 以及其他各种类型的旧样式的 csproj 文件迁移成新样式的 csproj 文件 - walterlv](https://walterlv.github.io/post/introduce-new-style-csproj-into-net-framework.html )

请看最简单创建一个 UWP 项目的代码

```csharp
<Project Sdk="MSBuild.Sdk.Extras/1.5.4">
  <PropertyGroup>#
    <TargetFrameworks>uap10.0.16299;</TargetFrameworks>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="MSBuild.Sdk.Extras" Version="1.5.4" />
  </ItemGroup>
</Project>
```

这里是使用多个平台，使用最低版本 16299 的原因是需要支持 dotnet standard

如果创建的项目是用来发布 nuget 的，那么就需要做一些设置，在继续阅读文本，我希望大家先看[项目文件中的已知 NuGet 属性（使用这些属性，创建 NuGet 包就可以不需要 nuspec 文件啦） - walterlv](https://walterlv.github.io/post/known-nuget-properties-in-csproj.html )

## 添加注释

如果需要在发布的 dll 添加 文档注释，那么请加下面代码

```csharp
   <PropertyGroup>
    <DocumentationFile>bin\$(Configuration)\$(TargetFramework)\$(AssemblyName).xml</DocumentationFile>
  </PropertyGroup>
```

所有的下面的代码都是放在 Project 标签里

```csharp
<Project Sdk="MSBuild.Sdk.Extras/1.5.4">
  <PropertyGroup>
    <TargetFrameworks>uap10.0.16299;</TargetFrameworks>
  </PropertyGroup>

  <PropertyGroup>
    <DocumentationFile>bin\$(Configuration)\$(TargetFramework)\$(AssemblyName).xml</DocumentationFile>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="MSBuild.Sdk.Extras" Version="1.5.4" />
  </ItemGroup>
</Project>
```

因为 `bin\$(Configuration)\$(TargetFramework)` 可以使用 `$(OutputPath)` 替换，上面的代码可以修改为

```csharp
   <PropertyGroup>
    <DocumentationFile>$(OutputPath)\$(AssemblyName).xml</DocumentationFile>
  </PropertyGroup>
```

## 防止警告生成的文件

一些生成的文件会让 VisualStudio 编译时警告，使用下面代码可以让 VisualStudio 不分析生成的文件

```csharp
<Target Name="PragmaWarningDisablePrefixer" AfterTargets="MarkupCompilePass2">
	<ItemGroup>
		<GeneratedCSFiles Include="**\*.g.cs;**\*.g.i.cs" />
	</ItemGroup>
	<Message Text="CSFiles: @(GeneratedCSFiles->'&quot;%(Identity)&quot;')" />
	<Exec Command="for %%f in (@(GeneratedCSFiles->'&quot;%(Identity)&quot;')) do echo #pragma warning disable &gt; %%f.temp &amp;&amp; type %%f &gt;&gt; %%f.temp &amp;&amp; move /y %%f.temp %%f" />
</Target>
```

## 自动添加版本

```csharp
  <PropertyGroup>
    <Build>$([System.DateTime]::op_Subtraction($([System.DateTime]::get_Now().get_Date()),$([System.DateTime]::new(2000,1,1))).get_TotalDays())</Build>
    <Revision>$([MSBuild]::Divide($([System.DateTime]::get_Now().get_TimeOfDay().get_TotalSeconds()), 2).ToString('F0'))</Revision>
    <Version>2.1.0.$(Revision)</Version>
  </PropertyGroup>
```

这样就可以自动添加版本号，虽然生成的版本号是用时间生成

这样的用法请看[项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://walterlv.github.io/post/known-properties-in-csproj.html )

如果只是想添加打包的版本号，请使用下面的代码

```csharp
 <PropertyGroup>
    <Build>$([System.DateTime]::op_Subtraction($([System.DateTime]::get_Now().get_Date()),$([System.DateTime]::new(2000,1,1))).get_TotalDays())</Build>
    <Revision>$([MSBuild]::Divide($([System.DateTime]::get_Now().get_TimeOfDay().get_TotalSeconds()), 2).ToString('F0'))</Revision>
    <Version>2.1.0</Version> 
    <PackageVersion>2.1.5.$(Revision)</PackageVersion>
  </PropertyGroup>

```

打包的版本号是 PackageVersion ，项目版本号是 Version ，在打包的时候，找不到 PackageVersion 会自动使用 Version ，所以如果需要项目版本号和打包版本号不相同，就定义 Version 和  PackageVersion 使用不同的值。

但是很多小伙伴都是设置打包的版本号和项目版本号相同，这样如果有人说某个nuget出现问题，可以很快找到是哪里的问题。或者发布出去的包，可以通过查看 dll 的版本号就知道是哪个 Nuget 发布，因为 dll 的版本号和 nuget 的相同。

参见：[Roland Weigelt - How to Disable Warnings in Generated C# Files of UWP Apps](https://weblogs.asp.net/rweigelt/disable-warnings-in-generated-c-files-of-uwp-app )

![](http://image.acmx.xyz/lindexi%2F20186112028468851.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
