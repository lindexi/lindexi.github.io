# Roslyn 打包 NuGet 包 BuildTransitive 文件夹用于穿透依赖传递拷贝文件

默认的 PackageReference 可以实现传递依赖，传递依赖的含义是是假定 B 项目安装了 A 库，而 C 项目依赖 B 项目，那么 C 项目将会自然拿到 A 库的 DLL 引用。但默认的 NuGet 包的构建指导文件 targets 命令是不会在传递执行的，也就是如上的 C 项目将不会执行 B 项目安装的 A 库里面的 target 内容

有一些项目需要拷贝自定义文件，例如拷贝图片或者一些 Native 的 DLL 等资源。如 WPF 框架需要拷贝 PenIME 等资源。如果只是在最底层的项目安装了库，那为了让可执行文件项目也输出库的资源，就需要在可执行项目上也安装库。以上的方法的不足在于安装复杂，也许会忘记安装

本文告诉大家一个解决方法是通过在制作库的时候，加上 BuildTransitive 文件夹，在此文件夹内添加构建指导文件，此时这个构建指导文件 targets 文件里面的命令将会在传递中执行，也就是说只需要在底层的项目安装即可，不需要在可执行项目上也安装库

<!--more-->
<!-- CreateTime:2021/7/15 19:40:39 -->

<!-- 发布 -->
<!-- 标签：Roslyn,MSBuild,编译器,nuget,打包 -->

我写了很多 Rolsyn 的关于打包 NuGet 包相关的博客，如 [Roslyn 打包自定义的文件到 NuGet 包](https://blog.lindexi.com/post/Roslyn-%E6%89%93%E5%8C%85%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%87%E4%BB%B6%E5%88%B0-NuGet-%E5%8C%85.html ) 这一篇。在这个博客告诉大家如何打包自定义的文件到 NuGet 包，和将自定义的文件放在输出文件夹里面

但是以上方法存在的问题如上文，只有制定了 PackageReference 安装此库的项目，才能将自定义的文件输出。其他的项目，如果只是做传递引用，没有直接在 csproj 上写 PackageReference 安装此库，那么不会拿到自定义输出文件

如果此时自定义的文件是 Native 的 DLL 资源，而只有在底层的库安装了库，那就会让可执行项目输出文件夹没有这些 Native 的 DLL 内容，在运行的时候找不到 Native 的 DLL 文件

为了解决此问题，咱可以将原先写在 Build 文件夹下的 targets 等文件，修改放在 NuGet 包的 BuildTransitive 文件夹。除此之外，其他逻辑都和之前相同

也就是说如果期望在引用传递中，每个引用的项目都会执行到 NuGet 里面的 targets 和 props 文件的命令，只需要将 targets 和 props 文件放入到 BuildTransitive 文件夹即可。其他逻辑都和之前相同

敲黑板，只需要将原本放在 NuGet 里面的 Build 文件夹，重命名为 BuildTransitive 即可

以下是告诉大家如何制作的方法，以及更多细节。本文的代码可以在本文的最后拿到，建议大家试一下本文的测试代码

回顾一下通过 csproj 创建 NuGet 包的方法，如 [Roslyn 打包自定义的文件到 NuGet 包](https://blog.lindexi.com/post/Roslyn-%E6%89%93%E5%8C%85%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%87%E4%BB%B6%E5%88%B0-NuGet-%E5%8C%85.html ) 的方法，咱创建一个叫 BuildTransitivePackage 的控制台项目，为了加上指导构建的 targets 等文件，咱需要新建一个 Build 文件夹，在里面放两个文件，文件名如下

- Package.props
- Package.targets

为了方便演示，咱来存放一张 Xamarin 的启动图片作为资源文件。期望是在软件的输出文件夹里面可以找到这张图片，换句话说就是这张图片需要放入到 NuGet 包中，接着输出到安装了此库的项目的输出文件夹

此图片文件放在 Assets 文件夹下，默认的启动图片是 SplashScreen.scale-400.png 文件

- `Assets\SplashScreen.scale-400.png`

接下来是本文的重点，咱来新建 BuildTransitive 文件夹来测试此功能，同样放入 targets 和 props 文件

- `BuildTransitive\Package.props`
- `BuildTransitive\Package.targets`

此时的项目文件大概如下

```
│  BuildTransitivePackage.csproj
│  BuildTransitivePackage.sln
│  Program.cs
│
├─Assets
│      SplashScreen.scale-400.png
│
├─Build
│      Package.props
│      Package.targets
│
└─BuildTransitive
        Package.props
        Package.targets
```

根据 NuGet 的规则，需要让放在 Build 文件夹的 targets 和 props 文件的命名和 NuGet 包的命名相同，咱可以在 csproj 上做一些更改，让以上的 targets 和 props 文件放入到 NuGet 包

```xml
  <ItemGroup>
    <None Include="Build\Package.targets" Pack="True" PackagePath="\build\$(PackageId).targets" />
    <None Include="Build\Package.props" Pack="True" PackagePath="\build\$(PackageId).props" />
  </ItemGroup>
```

同理对 BuildTransitive 文件夹的 targets 和 props 文件也输出到 NuGet 包

```xml
  <ItemGroup>
     <None Include="BuildTransitive\Package.targets" Pack="True" PackagePath="\buildTransitive\$(PackageId).targets" /> 
     <None Include="BuildTransitive\Package.props" Pack="True" PackagePath="\buildTransitive\$(PackageId).props" /> 
  </ItemGroup>
```

接下来将图片文件也打包放入到 NuGet 文件里面，为了方便测试，咱将图片复制两次，作为 Image1.png 和 Image2.png 两个文件。此时就可以测试两个不同的方法，分别是在 Build 文件夹和在 BuildTransitive 文件夹里面的 Package.targets 拷贝这两个文件。此时可以通过输出文件夹里面能找到哪些 png 文件来了解，放在 Build 文件夹和在 BuildTransitive 文件夹里面的构建命名的执行

```xml
    <None Include="Assets\SplashScreen.scale-400.png" Pack="True" PackagePath="\content\Image1.png" />
    <None Include="Assets\SplashScreen.scale-400.png" Pack="True" PackagePath="\content\Image2.png" />
```

修改之后的 csproj 文件内容如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net5.0</TargetFramework>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
    <Version>1.0.19</Version>
  </PropertyGroup>

  <ItemGroup>
    <None Include="Assets\SplashScreen.scale-400.png" Pack="True" PackagePath="\content\Image1.png" />
    <None Include="Assets\SplashScreen.scale-400.png" Pack="True" PackagePath="\content\Image2.png" />
    <None Include="Build\Package.targets" Pack="True" PackagePath="\build\$(PackageId).targets" />
    <None Include="Build\Package.props" Pack="True" PackagePath="\build\$(PackageId).props" />

     <None Include="BuildTransitive\Package.targets" Pack="True" PackagePath="\buildTransitive\$(PackageId).targets" /> 
     <None Include="BuildTransitive\Package.props" Pack="True" PackagePath="\buildTransitive\$(PackageId).props" /> 
  </ItemGroup>
</Project>
```

接下来在 `Build\Package.targets` 添加如下代码，用于将 Image1.png 文件输出到安装了库的项目的输出文件夹

```xml
<Project>
  <Target Name="CopyImage1File" BeforeTargets="AfterCompile">
     <Copy SourceFiles="$(MSBuildThisFileDirectory)..\content\Image1.png" 
           DestinationFolder="$(OutputPath)" SkipUnchangedFiles="True" />
  </Target>
</Project>
```

同理在 `BuildTransitive\Package.targets` 添加如下代码，用于将 Image2.png 拷贝到输出文件夹

```xml
<Project>
  <Target Name="CopyImage2File" BeforeTargets="AfterCompile">
    <Copy SourceFiles="$(MSBuildThisFileDirectory)..\content\Image2.png"
          DestinationFolder="$(OutputPath)" SkipUnchangedFiles="True" />
  </Target>
</Project>
```

打包 BuildTransitivePackage 项目，可以看到输出的 NuGet 包里面的文件结构如下

```
│  BuildTransitivePackage.nuspec
│  [Content_Types].xml
│
├─build
│      BuildTransitivePackage.props
│      BuildTransitivePackage.targets
│
├─buildTransitive
│      BuildTransitivePackage.props
│      BuildTransitivePackage.targets
│
├─content
│      Image1.png
│      Image2.png
│
├─lib
│  └─net5.0
│          BuildTransitivePackage.dll
│          BuildTransitivePackage.runtimeconfig.json
│
├─package
│  └─services
│      └─metadata
│          └─core-properties
│                  3dbfc6f3466b4a8a9662add17409b308.psmdcp
│
└─_rels
        .rels
```

接着新建两个测试使用的项目，分别是 `BuildTransitivePackage.Foo1` 和 `BuildTransitivePackage.Foo2` 项目

让 `BuildTransitivePackage.Foo1` 项目安装 BuildTransitivePackage 的 NuGet 库，让 `BuildTransitivePackage.Foo2` 项目引用 `BuildTransitivePackage.Foo1` 项目

可以看到 `BuildTransitivePackage.Foo1` 项目的 csproj 文件内容大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net5.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="BuildTransitivePackage" Version="1.0.19" />
  </ItemGroup>

</Project>
```

而 `BuildTransitivePackage.Foo2` 项目的 csproj 文件内容大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net5.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\BuildTransitivePackage.Foo1\BuildTransitivePackage.Foo1.csproj" />
  </ItemGroup>

</Project>
```

以上的 `BuildTransitivePackage.Foo2` 没有通过 PackageReference 安装 BuildTransitivePackage 库。但是通过传递依赖，依然可以访问到 BuildTransitivePackage 库里面的 DLL 定义类型

此时尝试生成一下 `BuildTransitivePackage.Foo2` 然后去到输出文件夹

可以在输出文件夹找到 `Image2.png` 文件，证明放在 BuildTransitive 文件夹的 Package.targets 文件有在执行

但是没有找到 `Image1.png` 文件，证明放在 Build 文件夹的 Package.targets 文件是没有被执行，符合预期

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1658ceb689b8285242e1411e2104643409facbc7/BuildTransitivePackage) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1658ceb689b8285242e1411e2104643409facbc7/BuildTransitivePackage) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1658ceb689b8285242e1411e2104643409facbc7
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 BuildTransitivePackage 文件夹

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 

官方文档 [Allow package authors to define build assets transitive behavior · NuGet/Home Wiki](https://github.com/NuGet/Home/wiki/Allow-package--authors-to-define-build-assets-transitive-behavior )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
