
# Roslyn 解决找不到 SolutionDir 属性

在旧版本的 csproj 格式也就是 Franken-proj 格式，可以使用 SolutionDir 拿到当前sln文件所在的文件夹，但是在 SDK Style 格式的项目文件，是拿不到这个属性的，本文告诉大家如何做到兼容之前的逻辑

<!--more-->


<!-- CreateTime:2020/8/31 19:06:05 -->

<!-- 发布 -->

其实不是 SDK Style 拿不到 SolutionDir 属性，而是因为没有使用 sln 加入构建

如我在一个项目里面的 csproj 文件添加下面代码

```xml
  <Target Name="KofegaheqallwhikaDalbedalkihi" AfterTargets="BeforeBuild">
    <Warning Text="Solution $(SolutionDir)"/>
  </Target>
```

如果我使用 dotnet 命令构建这个 csproj 文件，那么将会输出如下代码

```csharp
 warning : Solution *Undefined*
```

如果我使用 dotnet 命令构建 sln 文件，那么可以输出 sln 所在的文件夹

这就是不同点

但是如果我只是想通过 dotnet 或 msbuild 命令构建 csproj 文件，而不构建 sln 文件，此时还需要使用 SolutionDir 属性。那么请在 sln 文件夹添加一个叫 Directory.Build.props 文件，这个文件可以设置这个文件夹里面的所有项目的构建

而 `$(SolutionDir)` 属性的含义也就是一个属性，因此只需要返回当前 sln 所在文件夹就可以

```xml
<Project>
  <PropertyGroup>
    <SolutionDir>$(MSBuildThisFileDirectory)</PackageOutputPath>
  </PropertyGroup>
</Project>
```

上面代码的 `$(MSBuildThisFileDirectory)` 就是获取当前文件所在的文件夹，而当前文件是放在和 sln 文件相同的文件夹，因此这个就是和 SolutionDir 等价

如果小伙伴不知道 Directory.Build.props 文件可以如何写，还请小伙伴看我的 [github](https://github.com/lindexi/lindexi_gd/tree/e6c7f883b16a18b1125a37b954580ff592e3807c/BaljuhairhearkelfereKakikunall) 仓库

关于 Directory.Build.props 文件 的作用请看 [Roslyn 使用 Directory.Build.props 文件定义编译](https://blog.lindexi.com/post/Roslyn-%E4%BD%BF%E7%94%A8-Directory.Build.props-%E6%96%87%E4%BB%B6%E5%AE%9A%E4%B9%89%E7%BC%96%E8%AF%91.html )

关于 `MSBuildThisFileDirectory` 等默认变量请看 [项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://blog.walterlv.com/post/known-properties-in-csproj.html )

更多编译相关博客请看 [手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。