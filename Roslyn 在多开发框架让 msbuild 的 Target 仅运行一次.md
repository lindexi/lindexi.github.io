# Roslyn 在多开发框架让 msbuild 的 Target 仅运行一次

在写预编译框架，因为安装项目会基于多个平台，也就是对应的 Target 会执行多次，而我需要的只是执行一次就可以

<!--more-->
<!-- CreateTime:2020/1/1 16:09:47 -->

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

这里的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7cd82e0726d1f0ca65011a1ad9ddd2f239fe43a3/WhibafagiLelfaihohe) 欢迎小伙伴访问

另一个方法是通过在 buildMultiTargeting 添加 targets 文件的方法。在 NuGet 的 build 文件夹和 buildMultiTargeting 的不同在于，在进行多框架编译时，将会在 build 的每个 Target 都执行，而 buildMultiTargeting 只是处理全部的框架的执行的逻辑。换句话说就是 build 里面的逻辑会对每个框架进行处理，而 buildMultiTargeting 里面的逻辑只是处理总的编译

在 NuGet 引用里面，可以包含 build 和 buildMultiTargeting 文件夹，两个文件夹里面的 Target 的调用会收到编译命令的影响。通过 buildMultiTargeting 文件夹里面儿 Target 只有多开发框架才会被调用的原理，可以在指定多开发框架时仅执行 buildMultiTargeting 里面的代码

如创建一个 Foo 的 NuGet 包，期望只有在编译时输出一次

在 build 文件夹里面创建 Foo.Targets 文件，在文件里面添加下面代码

```csharp
<Project>
  <Target Name="Foo" AfterTargets="Build" Condition="'$(TargetFrameworks)' == ''">
    <Message Text="林德熙是逗比"/>
  </Target>
</Project>
```

在 Foo.Targets 文件里面，如果时多开发框架，那么在判断 `'$(TargetFrameworks)' == '` 就会跳过执行，此时将会执行 `buildMultiTargeting` 的文件

在 buildMultiTargeting 文件夹里面创建 Foo.Targets 在这个文件里面添加下面代码

```csharp
<Project>
  <Target Name="Foo" AfterTargets="Build">
    <Message Text="林德熙是逗比"/>
  </Target>
</Project>
```

此时编译输出只有输出一次，也就是对应的 Target 只执行一次

如果在两个文件夹里面的 Foo.Targets 文件里面的 Target 相同代码太多，可以将相同的代码放在单独的文件夹，通过引用的方式，让对应的 Target 只调用一次

```csharp
|
|
--build
  |
  -- Foo.Targets
  |
  -- F.Targets
|
--buildMultiTargeting
  |
  -- Foo.Targets
```

如创建一个单独的 F.Targets 文件，将核心逻辑放在这个文件夹里面

```csharp
<Project>
  <Target Name="F">
        <Message Text="林德熙是逗比"/>
    </Target>
</Project>
```

在 build 和 buildMultiTargeting 通过引用这个文件减少重复代码

如 build 文件夹的代码，通过 DependsOnTargets 的方法调用引用的文件的 Target 方法

```csharp

<Project>
    <Import Project="F.targets"/>
  <Target Name="Foo" AfterTargets="Build"
            Condition="'$(TargetFrameworks)' == ''"
            DependsOnTargets="F">
    </Target>
</Project>
```

在 buildMultiTargeting 的代码

```xml
<Project>
    <Import Project="..\build\F.targets"/>
  <Target Name="Foo" AfterTargets="Build"
            DependsOnTargets="F">
    </Target>
</Project>
```

这个方法的代码比较复杂，我将代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/66ff7789cbbf7afe23b240708404a784a43b2eb6/WhibafagiLelfaihohe) 欢迎小伙伴访问

放在 github 的代码需要先用 VisualStudio 打开，右击 RanelwanemquHihaiyecewi 项目打包，此时可以在 `bin\debug` 文件夹找到 RanelwanemquHihaiyecewi.1.0.0.nupkg 文件，将 NuGet 设置使用本地文件夹的 NuGet 包，这样在另一个项目才能还原找到库是哪个。使用本地文件夹请看 [设置本地 NuGet 源](https://docs.microsoft.com/zh-cn/nuget/hosting-packages/local-feeds)

打开命令行在 LembowulalHiwhemjercurherwejem.csproj 文件所在的文件夹输入下面命令进行编译

```csharp
dotnet build -v n
```

此时可以看到在输出只有输出一次

在实际的项目 [SourceYard](https://github.com/dotnet-campus/SourceYard) 就用到这个方法，请看 [修改代码](https://github.com/dotnet-campus/SourceYard/pull/61)

[Targeting builds for multiple frameworks and machines](https://dev.to/davidwengier/targeting-builds-for-multiple-frameworks-and-machines-5h22 )

[Investigate configuration behavior when the values (Configurations/Platforms/TargetFrameworks) are conditioned or duplicated. · Issue #1829 · dotnet/project-system](https://github.com/dotnet/project-system/issues/1829 )

[Option to run target once per project in multi-targeting build · Issue #2781 · microsoft/msbuild](https://github.com/Microsoft/msbuild/issues/2781 )

[c# - How to make an MSBuild Target that only runs once instead of once, before Targets that run once per framework in the TargetFrameworks tag? - Stack Overflow](https://stackoverflow.com/questions/46675782/how-to-make-an-msbuild-target-that-only-runs-once-instead-of-once-before-target )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
