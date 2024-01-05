
# Roslyn 打包自定义的文件到 NuGet 包

在使用 sdk 格式的项目文件支持快速进行打包，但使用这个方式打包的时候将默认只带程序集输出文件，而没有带依赖的文件。本文告诉大家如何在打包的时候加上需要放在包里面的文件

<!--more-->


<!-- CreateTime:2019/12/18 20:08:32 -->

<!-- csdn -->

<!-- 标签：Roslyn,MSBuild,编译器,nuget,打包 -->

在 [VisualStudio 使用新项目格式快速打出 Nuget 包](https://blog.lindexi.com/post/VisualStudio-%E4%BD%BF%E7%94%A8%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E5%BF%AB%E9%80%9F%E6%89%93%E5%87%BA-Nuget-%E5%8C%85.html ) 告诉大家快速打包的方法，但有时候我需要将本地的一些资源或依赖也放在包里面，此时就需要用到下面的方法

在项目里面引用的资源，可以通过在引用的时候添加 Pack 属性设置打包，使用 PackagePath 属性设置打包的时候放在包里面的哪个文件夹

例如将项目里面引用的 `林德熙是逗比.txt` 打包放在 `lib\doubi` 文件夹里面，可以这样写

```xml
<None Include="林德熙是逗比.txt"
                  Pack="True"
                  PackagePath="\lib\doubi\" />
```

注意需要将 None 放在 ItemGroup 里面，请看代码

```xml
    <ItemGroup>
        <None Include="林德熙是逗比.txt"
              Pack="True"
              PackagePath="\lib\doubi\" />	
    </ItemGroup>
```

如果在这一行代码执行之前已经添加了引用这个文件，那么请将 Include 修改为 Update 请看下面代码

```xml
    <ItemGroup>
    	<None Include="*.txt"/>
    	<!-- 上面的代码使用 *.txt 包含了 林德熙是逗比.txt 文件，需要在下面代码使用更新 -->
        <None Update="林德熙是逗比.txt"
              Pack="True"
              PackagePath="\lib\doubi\" />	
    </ItemGroup>
```

而如果是引用了输出文件里面的某个 dll 如我引用了 Newtonsoft.Json.dll 这个库文件，我需要在 `bin\release` 文件夹里面引用文件，那么我将需要这样写

```xml
        <ItemGroup>
            <None Include="$(OutputPath)\net48\Newtonsoft.Json.dll"
                  Pack="True"
                  PackagePath="\tools\net48\Newtonsoft.Json.dll" />
        </ItemGroup>
```

上面代码将会在输出文件夹找到 Newtonsoft.Json.dll 将这个文件输出到打包文件夹里面

如果我是需要在运行过程引用的一些 C++ 运行库，那么同样可以上面方法

如果想要让打包到 NuGet 的文件，会输出到安装了此 NuGet 包的项目里面的输出路径下，可以添加 PackageCopyToOutput 属性，添加了此属性之后，将会输出到 content;contentFiles 文件夹里面，示例代码如下，更多请看 [Nuget 输出资源文件 - 唐宋元明清2188 - 博客园](https://www.cnblogs.com/kybs0/p/17943302 )

```xml
    <ItemGroup>
        <Content Include="vcomp140d.dll">
            <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
            <PackageCopyToOutput>true</PackageCopyToOutput>
        </Content>
    </ItemGroup>
```

另外在输出的时候也支持改名，例如在写 NuGet 的时候，在修改编译过程的 targets 和 props 文件是需要跟随包的名才能被执行。例如在 [Roslyn 通过 Target 修改编译的文件](https://blog.lindexi.com/post/Roslyn-%E9%80%9A%E8%BF%87-Target-%E4%BF%AE%E6%94%B9%E7%BC%96%E8%AF%91%E7%9A%84%E6%96%87%E4%BB%B6.html ) 写到的替换编译文件，此时要求对应的文件有规定的命名

在 NuGet 里面，要求执行的 targets 文件必须满足命名要求，需要命名为 `NuGet包id.targets` 才会被执行，对应的 props 文件也相同

如果是自己手写文件名，在更改 NuGet 包 id 的时候如果没有更改，或复制不对，那么会发现没有执行

简单的解决方法是在打包的时候自动修改对应的文件包

先在项目文件的 build 文件夹里面添加 package.targets 和 package.props 文件，在项目文件添加下面代码进行输出

```xml
<None Include="build\package.targets" Pack="True" PackagePath="\build\$(PackageId).targets" />
<None Include="build\package.props" Pack="True" PackagePath="\build\$(PackageId).props" />
```

这样在输出的时候就会自动更改文件名

如果某些依赖文件是另外的其他项目构建生成的，则可以将引用放入到 Target 里面，设置在 `_GetPackageFiles` 之前执行。比如说常用的依赖分析器项目，将分析器打入 NuGet 包里，大概的代码如下

```xml
  <ItemGroup>
    <!-- 引用分析器项目，设置 ReferenceOutputAssembly 表示不引用程序集，只是用来设置构建顺序 -->
    <ProjectReference Include="..\FooAnalyzer\FooAnalyzer.csproj" ReferenceOutputAssembly="false" />
  </ItemGroup>

  <!-- 添加分析器 -->
  <Target Name="_IncludeAllDependencies" BeforeTargets="_GetPackageFiles">
    <ItemGroup>
      <None Include="..\FooAnalyzer\bin\$(Configuration)\netstandard2.0\**" Pack="True" PackagePath="analyzers\dotnet\cs" />
    </ItemGroup>
  </Target>
```

以上代码通过 ProjectReference 设置项目的构建顺序，通过设置 ReferenceOutputAssembly 表示不引用程序集，详细请参阅 [三种方法设置 .NET/C# 项目的编译顺序，而不影响项目之间的引用 - walterlv](https://blog.walterlv.com/post/affects-project-building-order.html )

设置 `BeforeTargets="_GetPackageFiles"` 即可在 NuGet 打包收集文件之前，且在引用项目构建完成之后进行执行，从而可以添加引用。这里不能通过 `GenerateNuspecDependsOn` 的方式加上 Target 决定顺序，在 `C:\Program Files\dotnet\sdk\x.x.x\Sdks\NuGet.Build.Tasks.Pack\buildCrossTargeting\NuGet.Build.Tasks.Pack.targets` 里面配置了 `GenerateNuspecDependsOn` 将在 `_GetPackageFiles` 之后执行，如此将让 Target 里面的内容没有被引用

更多关于分析器请参阅 [使用 Source Generator 在编译你的 .NET 项目时自动生成代码 - walterlv](https://blog.walterlv.com/post/generate-csharp-source-using-roslyn-source-generator )

以上就是在 NuGet 包里打入自定义文件的方法，如果想要将 NuGet 包里的文件拷贝输出，则可以采用如下方法

在 package.targets 文件让对应的放在 NuGet 文件的资源输出，通过 [Copy](https://blog.lindexi.com/post/Roslyn-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-MSBuild-Copy-%E5%A4%8D%E5%88%B6%E6%96%87%E4%BB%B6.html) 的方式输出

先定义一个 Target 可以在编译完成之后输出

```xml
<Project>
    <Target Name="CopyXxxFile" AfterTargets="AfterBuild">

    </Target>
</Project>
```

请将 Target 的名修改为实际使用的复制文件

```
    <Target Name="_CopyXxxFile" AfterTargets="AfterBuild">
        <Copy SourceFiles="$(MSBuildThisFileDirectory)..\tools\nuget.exe" DestinationFiles="$(OutputPath)\tools\nuget.exe" SkipUnchangedFiles="True"></Copy>
    </Target>
```

使用 `$(MSBuildThisFileDirectory)` 拿到当前文件的文件夹，此时通过上一层文件就可以拿到 NuGet 包的文件夹。获取对应的文件进行输出到软件编译输出文件夹

关于文件复制请看 [Roslyn 如何使用 MSBuild Copy 复制文件](https://blog.lindexi.com/post/Roslyn-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-MSBuild-Copy-%E5%A4%8D%E5%88%B6%E6%96%87%E4%BB%B6.html)

如果这个库文件只是需要添加资源文件，不需要加上 lib 文件，也就是不添加引用，那么请设置这个项目作为工具库

```
    <IsTool>true</IsTool>
    <NoPackageAnalysis>true</NoPackageAnalysis>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
    <NoBuild>true</NoBuild>
    <IncludeBuildOutput>false</IncludeBuildOutput>
```

通过 `IsTool` 将不会在安装的项目引用编译的文件，也就是这个 NuGet 库只是工具，里面的 dll 不会被引用

在 NuGet 包嵌入 符号文件 的方法是添加 AllowedOutputExtensionsInPackageBuildOutputFolder 属性

```xml
<Project Sdk="Microsoft.NET.Sdk">
 <PropertyGroup>
    <!-- Include symbol files (*.pdb) in the built .nupkg -->
    <AllowedOutputExtensionsInPackageBuildOutputFolder>$(AllowedOutputExtensionsInPackageBuildOutputFolder);.pdb</AllowedOutputExtensionsInPackageBuildOutputFolder>
  </PropertyGroup>
</Project>
```

默认符号文件是放在 snupkg 文件，而不是放在 nupkg 文件，原因是将符号文件放在 nupkg 文件，会让 nupkg 文件太大。如果将符号文件放在 nupkg 文件，那么不需要开发者另外配置符号服务器，就可以拿到符号文件

官方文档 [NuGet 和 .NET 库](https://docs.microsoft.com/zh-cn/dotnet/standard/library-guidance/nuget ) 里明确告诉小伙伴请考虑将符号作为符号包 (`*.snupkg`) 发布到 NuGet.org 而不是放在 nupkg 文件，符号包 (`*.snupkg`) 为开发人员提供了良好的按需调试体验，而不会使主程序包大小膨胀，也不会影响那些不打算调试 NuGet 包的用户的还原性能

如果只是需要放注释文档，请看 [Roslyn 在 NuGet 包中放注释 xml 文件的方法](https://blog.lindexi.com/post/Roslyn-%E5%9C%A8-NuGet-%E5%8C%85%E4%B8%AD%E6%94%BE%E6%B3%A8%E9%87%8A-xml-%E6%96%87%E4%BB%B6%E7%9A%84%E6%96%B9%E6%B3%95.html)

详细请看 [NuGet 命令行上传找不到 snupkg 文件](https://blog.lindexi.com/post/NuGet-%E5%91%BD%E4%BB%A4%E8%A1%8C%E4%B8%8A%E4%BC%A0%E6%89%BE%E4%B8%8D%E5%88%B0-snupkg-%E6%96%87%E4%BB%B6.html )

[Roslyn 使用 Target 替换占位符方式生成 nuget 打包](https://blog.lindexi.com/post/Roslyn-%E4%BD%BF%E7%94%A8-Target-%E6%9B%BF%E6%8D%A2%E5%8D%A0%E4%BD%8D%E7%AC%A6%E6%96%B9%E5%BC%8F%E7%94%9F%E6%88%90-nuget-%E6%89%93%E5%8C%85.html )

[如何编写基于 Microsoft.NET.Sdk 的跨平台的 MSBuild Target（附各种自带的 Task） - walterlv](https://blog.walterlv.com/post/write-msbuild-target.html#microsoftnetsdk-%E4%B8%BA%E6%88%91%E4%BB%AC%E6%8F%90%E4%BE%9B%E7%9A%84%E7%8E%B0%E6%88%90%E5%8F%AF%E7%94%A8%E7%9A%84-task )

[如何创建一个基于 MSBuild Task 的跨平台的 NuGet 工具包 - walterlv](https://blog.walterlv.com/post/create-a-cross-platform-msbuild-task-based-nuget-tool.html )

[Roslyn 如何在 Target 引用 xaml 防止文件没有编译](https://blog.lindexi.com/post/Roslyn-%E5%A6%82%E4%BD%95%E5%9C%A8-Target-%E5%BC%95%E7%94%A8-xaml-%E9%98%B2%E6%AD%A2%E6%96%87%E4%BB%B6%E6%B2%A1%E6%9C%89%E7%BC%96%E8%AF%91.html )

[Roslyn 通过 Target 修改编译的文件](https://blog.lindexi.com/post/Roslyn-%E9%80%9A%E8%BF%87-Target-%E4%BF%AE%E6%94%B9%E7%BC%96%E8%AF%91%E7%9A%84%E6%96%87%E4%BB%B6.html )

![](http://image.acmx.xyz/lindexi%2F20197917354626)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。