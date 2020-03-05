# Roslyn 获得 sln 文件所在的文件夹

我找了很久没有发现 SolutionDir 这个定义，所以只能通过一个不通用的方法找到

<!--more-->
<!-- CreateTime:2019/7/22 8:57:14 -->

<!-- csdn -->

<!-- 标签：Roslyn,MSBuild,编译器 -->


在之前的项目可以使用 PreBuildEvent 的方式指定编译之前事件，新的项目格式也可以支持这个方法，只是支持不是很好

我就遇到在 Jenkins 无法编译通过，因为 PreBuildEvent 指定的 `$(SolutionDir)` 是空

在新的项目格式，找了很久都没有找到 `$(SolutionDir)` 的定义和找到运行的 sln 文件的定义的方法

于是通过 [Directory.Build.props](https://lindexi.oschina.io/lindexi/post/Roslyn-%E4%BD%BF%E7%94%A8-Directory.Build.props-%E6%96%87%E4%BB%B6%E5%AE%9A%E4%B9%89%E7%BC%96%E8%AF%91.html ) 的方法找到 sln 文件

<!-- ![](image/Roslyn 获得 sln 文件所在的文件夹/Roslyn 获得 sln 文件所在的文件夹0.png) -->

在 sln 文件所在的文件夹添加 Directory.Build.props 文件，因为很多项目的 sln 都在项目的最外，所以通过这个方法找到 sln 是可以的，只是不通用

如我有一个项目 lindexi 这个项目的文件夹请看下图

```csharp
|   Directory.Build.props
|   lindexi.sln
|   README.md
|
\---src
        lindexi.csproj
```

在 Directory.Build.props 添加下面代码

```csharp
<Project>
  <PropertyGroup>
    <SolutionDir>$(MSBuildThisFileDirectory)</SolutionDir>
  </PropertyGroup>
</Project>

```

因为 `$(MSBuildThisFileDirectory)` 就是当前的文件的文件夹，这个文件和 sln 文件刚好在相同的文件夹，所以通过这个方法就可以获得 sln 所在的文件夹

[项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://walterlv.com/post/known-properties-in-csproj.html )

[MSBuild Well-known Item Metadata](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-well-known-item-metadata?view=vs-2017 )

[MSBuild Reserved and Well-known Properties](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-reserved-and-well-known-properties?view=vs-2017 )

更多请看 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html )

![](http://image.acmx.xyz/lindexi%2F2019123205745682)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
