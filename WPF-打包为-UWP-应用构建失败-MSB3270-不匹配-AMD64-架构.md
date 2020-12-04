
# WPF 打包为 UWP 应用构建失败 MSB3270 不匹配 AMD64 架构

在使用 dotnet core 3.1 的 WPF 打包为 UWP 应用的时候，如果没有设置 PublishProfiles 那么将会在构建 x64 提示所生成项目的处理器架构“AMD64”与引用的处理器架构“x86”不匹配

<!--more-->


<!-- CreateTime:2020/3/13 11:06:33 -->



在我使用下面命令打包的时候，如果我将 `Platform` 设置为 x86 那么什么问题都没有，如果我设置为 x64 就会发现构建失败，请看 [https://github.com/dotnet-campus/TranslationTool/runs/504702520?check_suite_focus=true](https://github.com/dotnet-campus/TranslationTool/runs/504702520?check_suite_focus=true)

```csharp
msbuild TranslationTool.sln /p:Platform=x64 /p:Configuration=Debug /p:UapAppxPackageBuildMode=StoreOnly /p:AppxBundle=Never /p:PackageCertificateKeyFile=TranslationTool.Package_TemporaryKey.pfx /p:PackageCertificatePassword="123"
```

可以看到的英文提示如下

```
(ResolveAssemblyReferences target) ->   
C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\MSBuild\Current\Bin\Microsoft.Common.CurrentVersion.targets(2106,5): error MSB3270: There was a mismatch between the processor architecture of the project being built "AMD64" and the processor architecture of the reference "D:\a\TranslationTool\TranslationTool\Code\TranslationTool.WPF\bin\x86\Debug\netcoreapp3.1\win-x86\TranslationTool.WPF.dll", "x86". This mismatch may cause runtime failures. Please consider changing the targeted processor architecture of your project through the Configuration Manager so as to align the processor architectures between your project and references, or take a dependency on references with a processor architecture that matches the targeted processor architecture of your project. [D:\a\TranslationTool\TranslationTool\Code\TranslationTool.Package\TranslationTool.Package.wapproj]
```

中文提示如下

```
(ResolveAssemblyReferences 目标) ->
  C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\MSBuild\Current\Bin\Microsoft.Common.CurrentVersion.ta
rgets(2106,5): error MSB3270: 所生成项目的处理器架构“AMD64”与引用“D:\lindexi\TranslationTool\Code\TranslationTool.WPF\bin
\x86\Debug\netcoreapp3.1\win-x86\TranslationTool.WPF.dll”的处理器架构“x86”不匹配。这种不匹配可能会导致运行时失败。请 考虑通过配置管理器更改您的项目的目标处理器架构，以使您的项目
与引用间的处理器架构保持一致，或者为引用关联一个与您的项目的目标处理器架构相符的处理器架构。
```

解决方法是添加 PublishProfiles 文件，请看 [dotnet-campus/TranslationTool@1650f7a](https://github.com/dotnet-campus/TranslationTool/commit/1650f7a9a12cc2a9df1595477f21a928507ea201)

原因是在使用 .NET Core 3.1 的桌面应用需要修改使用独立发布，也需要指定不同的文件夹

除了在 WPF 项目添加 PublishProfiles 文件，还需要在打包项目添加代码

```xml
    <ProjectReference Include="..\lindexi\林德熙博客.csproj">
      <SkipGetTargetFrameworkProperties>True</SkipGetTargetFrameworkProperties>
      <TrustLevel>Full</TrustLevel>
      <PublishProfile Condition="'$(Configuration)|$(Platform)'=='Release|x86'">Properties\PublishProfiles\SelfContainedWin86.pubxml</PublishProfile>
      <PublishProfile Condition="'$(Configuration)|$(Platform)'=='Release|x64'">Properties\PublishProfiles\SelfContainedWin64.pubxml</PublishProfile>
      <PublishProfile Condition="'$(Configuration)|$(Platform)'=='Debug|x86'">Properties\PublishProfiles\SelfContainedWin86Debug.pubxml</PublishProfile>
      <PublishProfile Condition="'$(Configuration)|$(Platform)'=='Debug|x64'">Properties\PublishProfiles\SelfContainedWin64Debug.pubxml</PublishProfile>
    </ProjectReference>
```

如果使用 msbuild 命令行打包桌面应用为 UWP 应用请看 

[使用 msbuild 命令行编译 UWP 程序](https://blog.lindexi.com/post/win10-uwp-%E4%BD%BF%E7%94%A8-msbuild-%E5%91%BD%E4%BB%A4%E8%A1%8C%E7%BC%96%E8%AF%91-UWP-%E7%A8%8B%E5%BA%8F.html)

如何使用 Github 的自动构建请看

[Continuous integration and deployment for desktop apps with GitHub Actions](https://devblogs.microsoft.com/dotnet/continuous-integration-and-deployment-for-desktop-apps-with-github-actions/#comment-4898 )

[microsoft/github-actions-for-desktop-apps: This repo contains a sample WPF application to demonstrate how to create CI/CD pipelines using GitHub Actions.](https://github.com/microsoft/github-actions-for-desktop-apps )

如何在 VS 打包请看

[UWP 打包 win32 应用 添加防火墙例外](https://blog.lindexi.com/post/UWP-%E6%89%93%E5%8C%85-win32-%E5%BA%94%E7%94%A8-%E6%B7%BB%E5%8A%A0%E9%98%B2%E7%81%AB%E5%A2%99%E4%BE%8B%E5%A4%96.html)

本文链接的是 Github Action 是在 Github 上自动构建的服务，可以用来持续集成，可以用来做 NuGet 包

[dotnet 部署 github 的 Action 进行持续集成](https://blog.lindexi.com/post/dotnet-%E9%83%A8%E7%BD%B2-github-%E7%9A%84-Action-%E8%BF%9B%E8%A1%8C%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90.html)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。