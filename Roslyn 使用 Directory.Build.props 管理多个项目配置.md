# Roslyn 使用 Directory.Build.props 管理多个项目配置

在一些大项目需要很多独立的仓库来做，每个仓库之间都会有很多相同的配置，本文告诉大家如何通过 Directory.Build.props 管理多个项目配置

<!--more-->
<!-- CreateTime:2019/5/21 11:35:52 -->

<!-- 标签：Roslyn,MSBuild,编译器 -->

在我的 [MVVM 框架](https://www.nuget.org/packages/lindexi.wpf.Framework/ )需要三个不同的库，一个是 Framework 另外两个是 WPF 和 UWP 这三个库有很多重复的配置，如版本号和作者信息。

在之前，我每次发布的时候，我都需要修改三个不同的仓库的版本号，将几个版本号升级到最新，但是这个方法如大家所见，效率是很低的。

在我的另外的博客 [Roslyn 通过 Nuget 管理公司配置](https://blog.lindexi.com/post/Roslyn-%E9%80%9A%E8%BF%87-Nuget-%E7%AE%A1%E7%90%86%E5%85%AC%E5%8F%B8%E9%85%8D%E7%BD%AE.html ) 和 [Roslyn 通过 nuget 统一管理信息](https://blog.lindexi.com/post/Roslyn-%E9%80%9A%E8%BF%87-nuget-%E7%BB%9F%E4%B8%80%E7%AE%A1%E7%90%86%E4%BF%A1%E6%81%AF.html ) 介绍了统一管理配置的优点。

但是很显然，我暂时无法使用 nuget 的方法统一几个仓库的配置，我需要一个新的方式。

在看到我之前的博客 [Roslyn 使用 Directory.Build.props 文件定义编译](https://blog.lindexi.com/post/roslyn-%E4%BD%BF%E7%94%A8-directory.build.props-%E6%96%87%E4%BB%B6%E5%AE%9A%E4%B9%89%E7%BC%96%E8%AF%91.html ) 可以知道，通过 Directory.Build.props 文件可以修改配置。

于是本渣就使用这个方法统一配置，我在项目的最外面添加了 Directory.Build.props 文件

<!-- ![](image/Roslyn 使用 Directory.Build.props 管理多个项目配置/Roslyn 使用 Directory.Build.props 管理多个项目配置0.png) -->

![](http://image.acmx.xyz/lindexi%2F2018920151353506)

我创建了 Build 文件夹，这个文件夹里面就放一个文件，这个文件就是管理版本号

```
Build\Version.props

<Project>
  <PropertyGroup>
    <Version>2.1.156</Version>
  </PropertyGroup>
</Project>
```

现在的问题是如何让 Framework 三个不同的仓库引用这个文件，使用这个文件的版本号

通过 Directory.Build.props 添加下面的代码就可以让 Framework 项目找到版本号

```
<Project>
  <Import Project="build\Version.props" />
</Project>
```

这样就可以导入版本号文件，也就是可以让 Framework 等项目引用这个文件

除了版本号也有很多的信息可以通过这个方式配置，如设置作者信息，作者的信息是属性需要使用下面代码

```
<Project>
  <Import Project="build\Version.props" />
  <PropertyGroup>
    <Authors>lindexi</Authors>
  </PropertyGroup>
</Project>
```

除了作者还有很多相同的信息，如仓库的路径，下面我就直接将所有可以用到的属性写出来，这样大家可以复制下面的代码到自己的 Directory.Build.props 通过修改信息定义自己的配置

```

<Project>
  <Import Project="build\Version.props" />
  <PropertyGroup>
    <OutputPath>$(MSBuildThisFileDirectory)bin\$(Configuration)</OutputPath>
    <!-- 打包的文件夹 -->
    <PackageOutputPath>$(OutputPath)</PackageOutputPath>
    <!-- 是否包括符号，默认为 false 设置为 true 会创建 xx.symbols.nupkg 文件 -->
    <IncludeSymbols>true</IncludeSymbols>
    <!-- 作者名 -->
    <Authors>lindexi</Authors>
    <!-- 是否在每次重新编译的时候创建 nuget 包 -->
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
    <!-- 在 nuget 设置的项目的 url 通过这个 url 可以在 nuget.org 看到项目主页 -->
    <PackageProjectUrl>https://github.com/lindexi/UWP/tree/master/uwp/src/Framework</PackageProjectUrl>
    <!-- 在 nuget 设置项目的仓库，通过这个 url 可以点击到源代码 -->
    <RepositoryUrl>https://github.com/lindexi/UWP/tree/master/uwp/src/Framework</RepositoryUrl>
    <!-- 版权，很多时候都是写 Copyright © 2018 公司, All Rights Reserved. -->
    <Copyright>MIT</Copyright>
    <!-- 让用户安装的时候看到的版权链接 -->
    <PackageLicenseUrl>https://github.com/lindexi/UWP/blob/master/LICENSE</PackageLicenseUrl>
    <!-- 在 nuget 设置标签 -->
    <PackageTags>WPF;MVVM;UWP</PackageTags>
  </PropertyGroup>

  <ItemGroup>
    <!--指定自己的在安装 nuget 时修改编译-->
    <!-- 添加的 README 文件 -->
    <None Include="$(MSBuildThisFileDirectory)README.md" Pack="True" PackagePath="" Visible="false"/>
  </ItemGroup>

</Project>
```

上面的代码的 `$(MSBuildThisFileDirectory)` 就是替换文件所在的文件夹路径，更多请看 [项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://walterlv.com/post/known-properties-in-csproj.html )

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html ) 

参见：[Roslyn 入门 - CSDN博客](https://blog.csdn.net/column/details/23159.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
