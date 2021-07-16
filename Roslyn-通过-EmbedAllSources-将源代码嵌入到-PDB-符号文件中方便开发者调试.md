
# Roslyn 通过 EmbedAllSources 将源代码嵌入到 PDB 符号文件中方便开发者调试

咱造了一个轮子，咱可以非常方便将这个轮子库作为 NuGet 发布出去，造福其他开发者，或者毒害其他开发者。为什么说是毒害呢？因为有时候这个库存在坑，此时使用这个库的开发者就受到了伤害。为了安抚脆弱的开发者们，咱可以提高一下开发者们的调试效率，例如让开发者们可以调试到库里面的源代码
本文来告诉大家如何在项目文件里面添加上 EmbedAllSources 属性，将自己的代码嵌入到 PDB 符号文件里面，让开发者们在调试的时候，可以看到库的源代码

<!--more-->


<!-- 发布 -->
<!-- 标签：Roslyn,MSBuild,编译器,nuget,打包 -->

是否记得 PDB 符号文件的作用？符号文件将会记录着 DLL 文件里面的二进制内容和源代码之间的对应，根据 PDB 符号文件将可以关联上 DLL 和源代码。假设此时既有 PDB 符号文件，又有源代码，那自然就可以在调试的时候进入源代码进行调试

为了减少误解，必须说明的是，有 PDB 符号文件，可以方便进行代码关联调试。对于 .NET 的应用，没有 PDB 符号文件，依然可以愉快调试，原因是 .NET 的 DLL 里面自带了足量的信息。但是有 PDB 符号文件的话，调试更好评

默认的发布的库，都是不会带上源代码的，此时空有 PDB 符号文件，还不能愉快调试源代码。好在咱可以使用 EmbedAllSources 属性，将源代码嵌入到 PDB 符号文件里面，此时在 VisualStudio 2019 调试，将可以通过 PDB 文件调试进入到对应的源代码

将源代码放入到 PDB 符号文件的方法很简单，只需要在 csproj 项目文件添加如下代码即可

```xml
    <EmbedAllSources>true</EmbedAllSources>
```

默认的 NuGet 包是不带 PDB 符号文件的，官方推荐将 PDB 符号文件打到 snupkg 里面再推送。如果期望只作为一个 NuGet 包，可以加上如下代码也将符号文件放入 NuGet 里面。使用下面代码可以将 PDB 符号文件放入 NuGet 包，也将源代码打包压缩嵌入在 PDB 符号文件里面

```xml
  <PropertyGroup>
    <EmbedAllSources>true</EmbedAllSources>
    <!-- Include symbol files (*.pdb) in the built .nupkg -->
    <AllowedOutputExtensionsInPackageBuildOutputFolder>$(AllowedOutputExtensionsInPackageBuildOutputFolder);.pdb</AllowedOutputExtensionsInPackageBuildOutputFolder>
  </PropertyGroup>
```

更改之后的 csproj 项目文件内容大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

  <PropertyGroup>
    <TargetFrameworks>net45;netcoreapp3.1</TargetFrameworks>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
    <UseWpf>True</UseWpf>
    <UseWindowsForms>True</UseWindowsForms>
  </PropertyGroup>

  <PropertyGroup>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <EmbedAllSources>true</EmbedAllSources>
    <!-- Include symbol files (*.pdb) in the built .nupkg -->
    <AllowedOutputExtensionsInPackageBuildOutputFolder>$(AllowedOutputExtensionsInPackageBuildOutputFolder);.pdb</AllowedOutputExtensionsInPackageBuildOutputFolder>
  </PropertyGroup>
```

加上了 EmbedAllSources 就可以看到输出的 PDB 符号文件的体积比之前更大，原因是加上了源代码文件

在有嵌入源代码的符号文件，就可以在调试的时候，自动进入到源代码。但是和有源代码的调试不同的在于，此时的源代码是不支持更改逻辑的。另一个不足在于当前只支持 cs 等代码文件，还不支持 xaml 文件

此功能其实也是 SourceLink 的一个功能，需要 VisualStudio 2019 新版本才能支持。如果自己的代码是开源的，也可以通过 SourceLink 链接到 GitHub 等的方式，这样可以减少 NuGet 包的下载大小。详细请看 [dotnet 使用 SourceLink 将 NuGet 链接源代码到 GitHub 等仓库](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-SourceLink-%E5%B0%86-NuGet-%E9%93%BE%E6%8E%A5%E6%BA%90%E4%BB%A3%E7%A0%81%E5%88%B0-GitHub-%E7%AD%89%E4%BB%93%E5%BA%93.html )

其实将源代码放入 PDB 文件会比使用 SourceLink 链接到 GitHub 上更好，原因是从 GitHub 上拉代码的速度不如下一个大的 NuGet 文件

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。