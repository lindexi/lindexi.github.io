
# dotnet 配合 Gitlab 做自动推 Tag 时打包 NuGet 包

我现在的团队内部用的是 Gitlab 工具，在此工具上提供了 Gitlab CI CD 用于做自动化测试和构建。对于 CBB 来说，发布就是打出 NuGet 包然后上传到内部 NuGet 服务器。此时遇到的问题是，如何在 Gitlab 上执行打包，打包的时候如何指定 NuGet 包的版本号。因为 CBB 的特殊性，我要求每个 NuGet 正式发布的包都应该有一个对应的 Tag 号，这样将 NuGet 库安装到项目里面，之后发现问题了还能找到对应版本的代码
本文告诉大家如何配合 Gitlab 做自动推 Tag 时打包 NuGet 包。也就是本地打一个 Tag 号，推送到 Gitlab 上，就会出发 Gitlab 的自动构建，自动构建里面将会获取 Tag 版本号，然后打出 NuGet 包推送到服务器

<!--more-->


<!-- CreateTime:4/3/2020 8:31:45 AM -->



在阅读本文之前，期望大家了解什么是 Gitlab 的 Runner 以及如何开启，详细请看 [dotnet 配置 Gitlab 的 Runner 做 CI 自动构建](https://blog.lindexi.com/post/dotnet-%E9%85%8D%E7%BD%AE-Gitlab-%E7%9A%84-Runner-%E5%81%9A-CI-%E8%87%AA%E5%8A%A8%E6%9E%84%E5%BB%BA.html )

本文以一个 WPF 的库为例子，其他的 dotnet 库也是相同的方法

在 dotnet 里面打包的方法可以通过以下命令

```csharp
dotnet pack -c release
```

在 Gitlab 里面通过 Tag 出发构建命令可以在 `.gitlab-ci.yml` 添加下面代码

```yaml
tagpublish:
  stage: publish
  script:
    - "chcp 65001" # 这里的 chcp 65001 是支持中文 GBK 解决命令行输出

  only:
    - tags
```

通过 only 里面设置 tags 就可以在推送 Tag 到 Gitlab 上自动触发构建

现在的问题是如何让推送的 Tag 的版本号作为 NuGet 包的版本号

在 Gitlab 里面将会在执行构建的时候注入环境变量，使用 Tag 打包的时候将可以找到 `CI_COMMIT_TAG` 这个环境变量，这个变量的内容就是对应的 Tag 的内容。如我推送到 1.0.0 的 Tag 那么这个 `CI_COMMIT_TAG` 的值就是 1.0.0 可以通过这个原理拿到推送的内容

在编译里面的设置，可以在 csproj 或 targets 等文件里面通过 `$(环境变量)` 的方式拿到对应的环境变量，而设置输出的 NuGet 的版本号可以通过 `Version` 属性，也就是可以在 csproj 里面添加下面代码

```xml
<Version>$(CI_COMMIT_TAG)</Version>
```

此时将会从环境变量获取 `CI_COMMIT_TAG` 设置为 Version 属性，这样就能让打出来的 NuGet 包和对应的 Tag 相同

此时的 `.gitlab-ci.yml` 大概代码如下，请根据你的实际需求更改

```yaml
stages:
  - publish

# 推Tag打包
tagpublish:
  stage: publish
  script:
    - "chcp 65001"
    - 'dotnet pack -c release'
    - 'nuget push bin\Release\*.nupkg'
  only:
    - tags

```

以上方法有缺陷是如果打的 Tag 是包含 v 开头的，如 `v1.0.0` 此时就不好玩了。另一个问题是如果想要本地打一个 NuGet 包，那才是糟心

于是我开源了一个工具 [dotnetCampus.TagToVersion](https://github.com/dotnet-campus/dotnetCampus.TagToVersion ) 这个工具能根据传入的值更改 Version.props 文件的内容，更改为传入的值的版本号

使用这个工具的步骤就相对多了一点了，我在内部做了一个 dotnet 模版，减少了这些步骤。因为是内部用的模版我也没有打算开源

通过三个步骤能使用 [dotnetCampus.TagToVersion](https://github.com/dotnet-campus/dotnetCampus.TagToVersion ) 这个工具，这个工具能解决在本地打出 NuGet 包的版本号问题，能解决带 v 的特殊的 Tag 的问题

步骤1是添加 Directory.Build.props 文件

什么是 Directory.Build.props 文件请看 [Roslyn 使用 Directory.Build.props 文件定义编译](https://blog.lindexi.com/post/Roslyn-%E4%BD%BF%E7%94%A8-Directory.Build.props-%E6%96%87%E4%BB%B6%E5%AE%9A%E4%B9%89%E7%BC%96%E8%AF%91.html )

这个文件的大概作用就是定义此文件夹，以及此文件夹的子文件夹的编译步骤。在 Directory.Build.props 文件里面制定 Version.prop 文件的路径

```xml
<Project>
  <Import Project="build\Version.props" />
  <PropertyGroup>
    <PackageOutputPath>$(MSBuildThisFileDirectory)bin\$(Configuration)</PackageOutputPath>
    <Authors>dotnet-campus</Authors>
    <Company>dotnet-campus</Company>
    <LangVersion>latest</LangVersion>
    <PackageRequireLicenseAcceptance>false</PackageRequireLicenseAcceptance>
    <Description>描述信息</Description>
    <Copyright>Copyright (c) 2020 dotnet-campus</Copyright>
    <PackageProjectUrl>https://github.com/dotnet-campus/dotnetCampus.SourceYard</PackageProjectUrl>
    <RepositoryUrl>https://github.com/dotnet-campus/dotnetCampus.SourceYard.git</RepositoryUrl>
    <RepositoryType>git</RepositoryType>
    <PackageTags>source;dotnet;nuget;msbuild</PackageTags>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
  </PropertyGroup>
</Project>
```

请根据你的实际项目添加更改上面代码

核心的代码是设置 Version.props 的路径，放在build文件夹里面

在 Version.props 文件添加下面代码

```xml
<Project>
  <PropertyGroup>
    <Version>1.0.0</Version>
  </PropertyGroup>
</Project>
```

此时的版本号可以自由填写，这样就能解决本地打 NuGet 包指定版本号的问题。同时解决了非配置管理员的其他开发者也需要了解学习什么是 `CI_COMMIT_TAG` 的问题

对于其他开发者，只有看到这个 Version.props 文件，同时这个文件里面没有其他需要学习的知识，只是知道在这里更改版本号就可以。这个方式对于配置管理员来说很重要，甚至能决定这项技术能否推进。对于大部分开发者是不需要关心，也不愿意了解这部分技术，更多的是想要提升开发的效率，而太多的杂项的配置知识将会降低开发的效率

我用一个例子说明上面的问题，我在 csproj 里面放下面这段代码。然后我告诉你，每次新建文件的时候都需要向 NafojaneKakoweebi 添加一个 o 不然新建的文件没有作用。假设你在我的团队里面，你要不要打我

```xml
<NafojaneKakoweebi>WhaljeahalqaboBembibocaooo</NafojaneKakoweebi>
```

当然，这个例子大家看看就好，虽然我真的写了这样的逻辑。通过仔细看还是能了解下面代码的含义，但是对于开发者的效率降低也确实存在

```xml
  <PropertyGroup>
    <Build>$([System.DateTime]::op_Subtraction($([System.DateTime]::get_Now().get_Date()),$([System.DateTime]::new(2000,1,1))).get_TotalDays())</Build>
    <Revision>$([MSBuild]::Divide($([System.DateTime]::get_Now().get_TimeOfDay().get_TotalSeconds()), 2).ToString('F0'))</Revision>
    <Version>1.0.0.$(Revision)</Version>
  </PropertyGroup>
```

通过本文的第二个方法，可以让开发者在开发的时候不会碰到配置管理的代码，开发者只是看到 Version.props 文件，这个文件有版本号。而开发者最多也就是改版本号做本地打包

那么对于配置管理来说，如何在推送 Tag 打包的时候自动设置版本号？通过 [dotnetCampus.TagToVersion](https://github.com/dotnet-campus/dotnetCampus.TagToVersion ) 这个工具根据  `CI_COMMIT_TAG` 更改 Version 文件的内容就可以做到

修改 `.gitlab-ci.yml` 为下面代码

```yaml
stages:
  - publish

# 推Tag打包
tagpublish:
  stage: publish
  script:
    - "chcp 65001"
    - "dotnet new tool-manifest"
    - "dotnet tool install dotnetCampus.TagToVersion"
    - "dotnet tool run dotnet-TagToVersion -t $CI_COMMIT_TAG"
    - 'dotnet pack -c release'
    - 'nuget push bin\Release\*.nupkg'
  only:
    - tags
```

对比开始的 `.gitlab-ci.yml` 文件可以看到添加了下面代码

```csharp
    - "dotnet new tool-manifest"
    - "dotnet tool install dotnetCampus.TagToVersion"
    - "dotnet tool run dotnet-TagToVersion -t $CI_COMMIT_TAG"
```

这三句话就是创建一个 dotnet tool 清单，安装 dotnet tool 工具，然后运行工具而已。如果提前在 gitlab runner 的设备上安装了 dotnetCampus.TagToVersion 那么可以将上面代码压缩为一句代码

```csharp
    - "dotnet TagToVersion -t $CI_COMMIT_TAG"
```

执行上面代码将会自动根据传入的参数修改 Version.props 的值，这样就能做到推送 Tag 打包对应版本号

用推 Tag 打包的好处是解决回滚代码的时候，需要用到某个 NuGet 包进行调试，可以找到对应版本的代码。同时解决了手动叫开发者打 NuGet 包的时候需要记得添加 Tag 号

在调试对应版本的 NuGet 的代码的时候，我推荐使用以下方法

- [Roslyn 让 VisualStudio 急速调试底层库方法](https://blog.lindexi.com/post/Roslyn-%E8%AE%A9-VisualStudio-%E6%80%A5%E9%80%9F%E8%B0%83%E8%AF%95%E5%BA%95%E5%B1%82%E5%BA%93%E6%96%B9%E6%B3%95.html )
- [VS DLL引用替换插件](https://github.com/dotnet-campus/DllReferencePathChanger )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。