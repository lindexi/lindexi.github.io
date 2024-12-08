# msbuild Roslyn 行为详解

本文来告诉大家 msbuild Roslyn 的行为，本文非新手友好

<!--more-->
<!-- CreateTime:2021/11/19 19:15:23 -->

## 常用参数

[项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://blog.walterlv.com/post/known-properties-in-csproj.html )

[项目文件中的已知 NuGet 属性（使用这些属性，创建 NuGet 包就可以不需要 nuspec 文件啦） - walterlv](https://blog.walterlv.com/post/known-nuget-properties-in-csproj )

## 常用判断

[msbuild 项目文件常用判断条件](https://blog.lindexi.com/post/msbuild-%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6%E5%B8%B8%E7%94%A8%E5%88%A4%E6%96%AD%E6%9D%A1%E4%BB%B6.html )

## 行为属性

### CopyLocalLockFileAssemblies

拷贝引用项到输出。默认行为下，对于 DLL 项目来说不拷贝引用项到输出，对于 Exe 或 WinExe 项目来说默认是会拷贝引用项到输出

如果想要让 DLL 项目也将引用项拷贝到输出，则可以配置 CopyLocalLockFileAssemblies 为 true 的值，如以下代码

```xml
  <PropertyGroup>
    <!-- 拷贝输出项，用于给 MauiWpfAdapt 项目引用 -->
    <CopyLocalLockFileAssemblies>true</CopyLocalLockFileAssemblies>
  </PropertyGroup>
```

## 多框架相关

### 调用次数

如有 Target 设置 `AfterTargets="Build"` 将在多框架下被分别调用，每个框架都会调用一次，最后还会再调用一次表示总的构建。调用次数等于框架数量加一

如在框架为 `<TargetFrameworks>net45;netcoreapp3.1;net6.0</TargetFrameworks>` 那将会分别在 `net45` `netcoreapp3.1` `net6.0` 调用一次，以及最终构建完成调用一次

### 多框架的 IntermediateOutputPath 属性值

默认是使用 IntermediateOutputPath 表示 `obj` 下的缓存文件夹，可以用来输出构建相关的缓存文件，在多框架下，默认是加上框架的路径，如 `obj\Debug\net45\` 和 `obj\Debug\net5.0\` 文件夹

<!-- 随着调用的次数，各个框架构建的时候，将会带上框架的路径。在最终构建，也就是总的框架构建，调用时的值是不带上具体的框架的，如 `obj\Debug\` 文件夹 -->

更具体而言，整个 Target 将会被多次调用，多次调用包含各个框架的各次调用，以及总的一次调用。各个框架的各次调用中，将会拼接上框架的路径。在总的一次调用中，不会带上具体的框架

测试逻辑如下

```xml
  <Target Name="GallikufawhaGebalule" AfterTargets="Build">
    <Warning Text="IntermediateOutputPath: $(IntermediateOutputPath)" />
  </Target>
```

在多框架 `<TargetFrameworks>net45;net5.0</TargetFrameworks>` 下，以上代码输出如下

```
1>C:\lindexi\Code\Foo.csproj(17,3): warning : IntermediateOutputPath: obj\Debug\net45\
1>已完成生成项目“Foo.csproj”的操作。
1>C:\lindexi\Code\Foo.csproj(17,3): warning : IntermediateOutputPath: obj\Debug\net5.0\
1>已完成生成项目“Foo.csproj”的操作。
1>C:\lindexi\Code\Foo.csproj(17,3): warning : IntermediateOutputPath: obj\Debug\
```

可见前面两次分别是 net45 和 net5.0 框架的构建，带上了框架路径。最后一次是总的调用，不带上任何框架路径

## NuGet 相关

### 配置属性大全

[dotnet 打包 NuGet 的配置属性大全整理](https://blog.lindexi.com/post/dotnet-%E6%89%93%E5%8C%85-NuGet-%E7%9A%84%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%A4%A7%E5%85%A8%E6%95%B4%E7%90%86.html )

### 动态加入打包到 NuGet 包的文件时机

可在 `_GetPackageFiles` 这个 Target 前执行，在此执行加入 Nuget 打包文件才是有效，在这个时机之后将会无效，如以下代码

```xml
  <ItemGroup>
    <None Include="build\package.targets" Pack="True" PackagePath="\build\$(PackageId).targets" />
  </ItemGroup>

  <Target Name="FooIncludeAllDependencies" BeforeTargets="_GetPackageFiles">
    <ItemGroup>
      <None Include="..\Foo\Foo.dll" Pack="True" PackagePath="analyzers\dotnet\cs" />
    </ItemGroup>
  </Target>
```

以上代码的两个加入打包的文件都会成功都被加入打包。更多请参阅 [Roslyn 打包自定义的文件到 NuGet 包](https://blog.lindexi.com/post/Roslyn-%E6%89%93%E5%8C%85%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%87%E4%BB%B6%E5%88%B0-NuGet-%E5%8C%85.html )

### 多框架的 BuildMultiTargeting 和 Build 文件夹下的 Target 调用次数

在 Build 文件夹下的 Target 将会在各个框架分别执行。放在 BuildMultiTargeting 的 Target 将只会执行一次，详细请看 [Roslyn 在多开发框架让 msbuild 的 Target 仅运行一次](https://blog.lindexi.com/post/Roslyn-%E5%9C%A8%E5%A4%9A%E5%BC%80%E5%8F%91%E6%A1%86%E6%9E%B6%E8%AE%A9-msbuild-%E7%9A%84-Target-%E4%BB%85%E8%BF%90%E8%A1%8C%E4%B8%80%E6%AC%A1.html )

可以同时存在 BuildMultiTargeting 和 Build 文件夹，里面的内容相互不干涉，除非有设置调用关系和引用

### 独立框架的 Target 定义属性给多框架使用

在 Build 文件夹下的 Target 将会在各个框架分别独立执行，而 BuildMultiTargeting 只会执行一次。如果在 Build 文件夹下定义属性，如下面代码

```xml
  <Target Name="BuildSourceNuGet" AfterTargets="Build">
    <PropertyGroup>
      <IntermediateOutputPathCombine>$(IntermediateOutputPathCombine);$(IntermediateOutputPath)</IntermediateOutputPathCombine>
    </PropertyGroup>

    <Warning Text="build once"/>
  </Target>
```

预期是各个框架在 `IntermediateOutputPathCombine` 属性上定义各自的 IntermediateOutputPath 路径。然而在 BuildMultiTargeting 下拿到的依然是空值

```xml
  <Target Name="BuildSourceNuGetMultiTargeting" AfterTargets="Build">
    <Warning Text="MultiTargetingBuild: $(IntermediateOutputPathCombine)"/>
  </Target>
```

因此不能在 BuildMultiTargeting 上使用到各个 Build 文件夹下的 Target 收集的属性内容

### 多框架下获取 TargetFrameworks 属性

可在各个框架构建时，获取到 `$(TargetFrameworks)` 属性内容，属性内容为全部框架。如以下测试项目

NuGet 包的 Package.targets 文件：

```xml
  <Target Name="HejurjeelodayJicochibiki" AfterTargets="Build">
    <Warning Text="TargetFrameworks=$(TargetFrameworks) | TargetFramework=$(TargetFramework)"/>
  </Target>
```

应用多框架项目：

```xml
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFrameworks>net6.0;net7.0;net9.0</TargetFrameworks>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
```

构建输出信息如下

```
1>JearjikunaBemnenerenehechekee -> C:\lindexi\Code\JearjikunaBemnenerenehechekee\bin\Debug\net7.0\JearjikunaBemnenerenehechekee.dll
1>C:\Users\lindexi\.nuget\packages\hekairkefairfallqecairwaqai\2.0.0\build\HekairkefairfallQecairwaqai.targets(4,5): warning : TargetFrameworks=net6.0;net7.0;net9.0 | TargetFramework=net7.0
1>已完成生成项目“JearjikunaBemnenerenehechekee.csproj”的操作。
1>JearjikunaBemnenerenehechekee -> C:\lindexi\Code\JearjikunaBemnenerenehechekee\bin\Debug\net6.0\JearjikunaBemnenerenehechekee.dll
1>C:\Users\lindexi\.nuget\packages\hekairkefairfallqecairwaqai\2.0.0\build\HekairkefairfallQecairwaqai.targets(4,5): warning : TargetFrameworks=net6.0;net7.0;net9.0 | TargetFramework=net6.0
1>已完成生成项目“JearjikunaBemnenerenehechekee.csproj”的操作。
1>JearjikunaBemnenerenehechekee -> C:\lindexi\Code\JearjikunaBemnenerenehechekee\bin\Debug\net9.0\JearjikunaBemnenerenehechekee.dll
1>C:\Users\lindexi\.nuget\packages\hekairkefairfallqecairwaqai\2.0.0\build\HekairkefairfallQecairwaqai.targets(4,5): warning : TargetFrameworks=net6.0;net7.0;net9.0 | TargetFramework=net9.0
```

以上测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/592c0272e08bceb0e608d09d2ca560724cdd8fae/Roslyn/HewurchawawjelfeLairkonokawhere) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/592c0272e08bceb0e608d09d2ca560724cdd8fae/Roslyn/HewurchawawjelfeLairkonokawhere) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 592c0272e08bceb0e608d09d2ca560724cdd8fae
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 592c0272e08bceb0e608d09d2ca560724cdd8fae
```

获取代码之后，进入 Roslyn/HewurchawawjelfeLairkonokawhere 文件夹，即可获取到源代码

继续进行测试。如 TargetFrameworks 只包含一项，如下面代码所示：

```xml
    <TargetFrameworks>net9.0</TargetFrameworks>
```

此时的输出警告信息如下

```
warning : TargetFrameworks=net9.0 | TargetFramework=net9.0
```

证明在 TargetFrameworks 只包含一项时，依然能够获取这一项

反着，如果不写 TargetFrameworks 属性，将其换成 TargetFramework 属性，如以下代码所示

```xml
<TargetFramework>net9.0</TargetFramework>
```

则此时的输出警告信息如下

```
warning : TargetFrameworks= | TargetFramework=net9.0
```

通过以上警告输出，可见将 TargetFrameworks 换成 TargetFramework 时，将很符合预期的不能获取到 `$(TargetFrameworks)` 属性内容

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )