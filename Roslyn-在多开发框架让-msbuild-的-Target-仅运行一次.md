
# Roslyn 在多开发框架让 msbuild 的 Target 仅运行一次

在写预编译框架，因为安装项目会基于多个平台，也就是对应的 Target 会执行多次，而我需要的只是执行一次就可以

<!--more-->


<!-- csdn -->
<!-- 标签：Roslyn,MSBuild,编译器 -->

创建一个控制台项目，修改项目文件，然后使用 dotnet build 可以看到 Foo 输出两次

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFrameworks>netcoreapp3.1;net45</TargetFrameworks>
  </PropertyGroup>

  <Target Name="Foo" AfterTargets="AfterBuild">
  	<Warning Text="Foo"></Warning>
  </Target>

</Project>

```

因为这是在两个平台分别输出，如果想要在编译只运行一次，可以基于以下

- DispatchToInnerBuilds
- PreBuildEvent
- GenerateNuspec

如下面代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFrameworks>netcoreapp3.1;net45</TargetFrameworks>
  </PropertyGroup>

  <Target Name="Foo" BeforeTargets="DispatchToInnerBuilds">
  	<Warning Text="Foo"></Warning>
  </Target>

</Project>

```

如果是在 GenerateNuspec 之后也就是需要用户创建 NuGet 库才能执行

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7cd82e0726d1f0ca65011a1ad9ddd2f239fe43a3/WhibafagiLelfaihohe) 欢迎小伙伴访问

[Targeting builds for multiple frameworks and machines](https://dev.to/davidwengier/targeting-builds-for-multiple-frameworks-and-machines-5h22 )

[Investigate configuration behavior when the values (Configurations/Platforms/TargetFrameworks) are conditioned or duplicated. · Issue #1829 · dotnet/project-system](https://github.com/dotnet/project-system/issues/1829 )

[Option to run target once per project in multi-targeting build · Issue #2781 · microsoft/msbuild](https://github.com/Microsoft/msbuild/issues/2781 )

[c# - How to make an MSBuild Target that only runs once instead of once, before Targets that run once per framework in the TargetFrameworks tag? - Stack Overflow](https://stackoverflow.com/questions/46675782/how-to-make-an-msbuild-target-that-only-runs-once-instead-of-once-before-target )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。