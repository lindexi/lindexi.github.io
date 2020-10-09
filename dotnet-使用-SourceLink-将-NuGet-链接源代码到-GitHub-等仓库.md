
# dotnet 使用 SourceLink 将 NuGet 链接源代码到 GitHub 等仓库

在发布 CBB 作为 NuGet 包的时候，我期望开发者在使用我的库进行调试，可以自动链接代码到对应打包的 GitHub 上的代码，可以从本地拿到对应的源代码进行调试。这样的调试方式对于开源项目来说，将会很方便

<!--more-->


<!-- CreateTime:2020/7/30 8:47:47 -->

<!-- 发布 -->

使用方法很简单，通过 SourceLink 就能做到。这个 SourceLink 是一系列的库，包含了给 Azure Devops 使用的和给 GitHub 使用的，给 Gitlab 使用的等

假定我现在是使用 GitHub 作为我的开源仓库，尝试我期望在 GitHub 的 Action 进行自动构建的时候，打包的 NuGet 包可以链接到本次打包的 commit 的代码，那么只需要安装 `Microsoft.SourceLink.GitHub` 库，同时额外添加一些配置就可以

打开 csproj 文件，添加以下代码

```xml
    <!-- 在 GitHub 的 Action 构建会添加 GITHUB_ACTIONS 变量 -->
    <PropertyGroup Condition="'$(GITHUB_ACTIONS)' == 'true'">
        <ContinuousIntegrationBuild>true</ContinuousIntegrationBuild>
        <!-- Optional: Publish the repository URL in the built .nupkg (in the NuSpec <Repository> element) -->
        <PublishRepositoryUrl>true</PublishRepositoryUrl>

        <!-- 只有在 GitHub 的 Action 构建才能使用源代码链接 -->
        <!-- 源代码链接需要使用 commit 号，而在 GitHub 的 Action 构建的 commit 才是对的 -->
        <!-- 本地构建，也许没有记得 commit 就构建，此时的 nuget 包的源代码是不对的，上传上去会让调试诡异 -->
        <!-- Optional: Embed source files that are not tracked by the source control manager in the PDB -->
        <EmbedUntrackedSources>true</EmbedUntrackedSources>

        <!-- 本地等不需要创建符号文件 -->
        <!-- Optional: Build symbol package (.snupkg) to distribute the PDB containing Source Link -->
        <IncludeSymbols>true</IncludeSymbols>
        <SymbolPackageFormat>snupkg</SymbolPackageFormat>
    </PropertyGroup>

    <ItemGroup Condition="'$(GITHUB_ACTIONS)' == 'true'">
        <PackageReference Include="Microsoft.SourceLink.GitHub" Version="1.0.0" PrivateAssets="All"/>
    </ItemGroup>
```

上面代码的 `PropertyGroup` 是额外的配置。为什么期望是在 `GITHUB_ACTIONS` 才进行 SourceLink 的安装？原因是假定没有在仅 GitHub 的 Action 自动构建时添加源代码链接，那么本地构建的时候也就自动添加了源代码链接。在本地构建的时候自动添加了源代码链接，也许本地的代码没有 commit 而此时拿到的 commit 也就不对了，或者本地 commit 了但是没有推送，然后也忘了推，那么开发者拿到的这个 NuGet 包将会找错 commit 或找不到。 其实找不到的问题没有找错的坑，因为开发者小伙伴也许因为 commit 找错了，而看到的不是实际运行的代码，接着就开始有趣的调试

在 GitHub 的 Action 自动构建时，将会添加 `GITHUB_ACTIONS` 变量，因此通过这个变量就可以了解当前是否在 GitHub 的 Action 构建

通过以上配置，可以看到打出的 NuGet 包的 nuspec 文件的 repository 属性加上了 commit 号

```xml
    <repository type="git" url="https://github.com/dotnet-campus/dotnetCampus.ClrAttachedProperty" commit="8308afac4666e0b002d66e04c82f97203e0b06a2" />
```

其实和没有使用 SourceLink 的 NuGet 包对比，这里的 nupkg 包仅更改了上面这一行

但是别忘了还有 snupkg 符号包，这个是做什么用的？小伙伴可以注意到在 nupkg 文件里面，这个压缩包文件里面是没有包含 pdb 符号文件的。在 dotnet 里面使用 pdb 符号文件是用来方便 VS 等工具进行调试，这个文件的作用就是告诉调试工具，对应的代码和二进制 dll 的关系

而在 NuGet 的 nupkg 包不包含 pdb 文件，而是将 pdb 文件放在 snupkg 包，这是为什么呢？原因是其实大量的开发者不关注调试本身，而是存在有大量的构建的还原，此时用不着 pdb 文件。此时如果将 pdb 文件放在 nupkg 包里面，将会让 nupkg 包的体积比较大，让还原速度降低，也就是下载 nupkg 的时间会比较多。因此就将 pdb 文件额外放在另一个 snupkg 文件里面，此时关注调试的开发者就可以在调试的时候拉 snupkg 文件，不关注调试的开发者就仅使用 nupkg 文件就可以

当然，小伙伴可以使用 AllowedOutputExtensionsInPackageBuildOutputFolder 属性在 NuGet 包嵌入符号文件 请看下面代码

```xml
<Project Sdk="Microsoft.NET.Sdk">
 <PropertyGroup>
    <!-- Include symbol files (*.pdb) in the built .nupkg -->
    <AllowedOutputExtensionsInPackageBuildOutputFolder>$(AllowedOutputExtensionsInPackageBuildOutputFolder);.pdb</AllowedOutputExtensionsInPackageBuildOutputFolder>
  </PropertyGroup>
</Project>
```

此时就不需要使用 snupkg 文件了，通过 nupkg 文件就可以包含符号文件了

更多关于符号文件请看 [NuGet 符号服务器](https://blog.lindexi.com/post/NuGet-%E7%AC%A6%E5%8F%B7%E6%9C%8D%E5%8A%A1%E5%99%A8.html)

为了让调试的时候和 GitHub 等仓库的源代码关联，此时就需要在创建的 pdb 文件做一点更改了，让 pdb 文件关联的文件是 GitHub 仓库的文件

因此在使用 SourceLink 是推荐添加 snupkg 包，将 snupkg 文件上传到 nuget 服务器

添加了 SourceLink 的 CBB 底层库就可以在使用的时候，调试可以在 VS 上在调用堆栈里面跳转到对应的 GitHub 的源代码。这里对 VS 版本有要求，需要使用 `Visual Studio 15.3+` 的版本，也就是 Visual Studio 2017 以上，才能使用源代码链接

本文的 SourceLink 在 GitHub 上完全开源: [https://github.com/dotnet/sourcelink](https://github.com/dotnet/sourcelink)

除了在 GitHub 上能用之外，还支持 Gitlab 等仓库，只需要修改对应的 NuGet 包

- GitHub: `<PackageReference Include="Microsoft.SourceLink.GitHub" Version="1.0.0" PrivateAssets="All"/>`
- Azure Repos(Visual Studio Team Services): `<PackageReference Include="Microsoft.SourceLink.AzureRepos.Git" Version="1.0.0" PrivateAssets="All"/>`
- Azure DevOps Server (former Team Foundation Server): `<PackageReference Include="Microsoft.SourceLink.AzureDevOpsServer.Git" Version="1.0.0" PrivateAssets="All"/>`
- GitLab: `<PackageReference Include="Microsoft.SourceLink.GitLab" Version="1.0.0" PrivateAssets="All"/>`
- gitweb: `<PackageReference Include="Microsoft.SourceLink.GitWeb" Version="1.1.0-beta-20204-02" PrivateAssets="All"/>`
- Bitbucket: `<PackageReference Include="Microsoft.SourceLink.Bitbucket.Git" Version="1.0.0" PrivateAssets="All"/>`

如果你只是期望发布源代码，可以通过 [SourceYard](https://github.com/dotnet-campus/SourceYard) 发布源代码的 NuGet 包，此时开发者安装 NuGet 包使用的不是二进制文件而是源代码文件，所有的源代码都能进行调试

使用方法是通过 NuGet 安装 `dotnetCampus.SourceYard` 库，或在 csproj 文件添加下面代码

```xml
    <ItemGroup>
        <PackageReference Include="dotnetCampus.SourceYard" Version="0.1.19099-alpha">
            <PrivateAssets>All</PrivateAssets>
        </PackageReference>
    </ItemGroup>
```

本文的 SourceYard 在 GitHub 上完全开源: [https://github.com/dotnet-campus/SourceYard](https://github.com/dotnet-campus/SourceYard)

使用 NuGet 能提升不少的开发效率，大部分功能的开发只需要三步。第一步安装 NuGet 库，第二步调用方法，第三步完成

如果安装之后构建失败，提示 error : SourceRoot items must include at least one top-level (not nested) item when DeterministicSourcePaths is true 构建失败，请在 Directory.Build.props 文件添加下面代码

```xml
<ItemGroup>
  <SourceRoot Include="$(MSBuildThisFileDirectory)/"/>
</ItemGroup>
```

详细请看 [dotnet 构建 SourceRoot items must include at least one top-level item when DeterministicSourcePaths is true 失败](https://blog.lindexi.com/post/dotnet-%E6%9E%84%E5%BB%BA-SourceRoot-items-must-include-at-least-one-top-level-item-when-DeterministicSourcePaths-is-true-%E5%A4%B1%E8%B4%A5.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。