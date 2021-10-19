# PublishFolderCleaner 让你的 dotnet 应用发布文件夹更加整洁

大家都知道，在 dotnet 发布时，将会在输出的 publish 文件夹包含所需的依赖。在 .NET Core 开始，引入了 AppHost 的概念，即使是单个程序集，也需要独立的 Exe 可执行文件带上实际包含 Main 函数的 dll 文件。特别是进行独立发布的时候，输出文件夹上有超级多个文件，看起来不清真。本文来告诉大家如何使用 PublishFolderCleaner 工具让发布文件夹只留一个 Exe 和一个 Lib 文件夹

<!--more-->
<!-- CreateTime:2021/10/18 8:31:15 -->

<!-- 发布 -->

## 使用方法

使用方法十分简单，只需要安装 [dotnetCampus.PublishFolderCleaner](https://www.nuget.org/packages/dotnetCampus.PublishFolderCleaner) 库即可。编辑入口项目的 csproj 文件，添加如下代码

```xml
  <ItemGroup>
    <PackageReference Include="dotnetCampus.PublishFolderCleaner" Version="3.0.3" />
  </ItemGroup>
```

接下来就和之前一样发布即可，不影响原有的发布步骤

## 效果

发布完成之后，打开发布文件夹，此时可以发现原本乱糟糟的文件夹被替换为只有一个 exe 可执行文件和一个 lib 文件夹。双击 exe 可执行文件即可获得和之前一样的效果

打开 Lib 文件夹，可以看到此文件夹里面就是原本放在发布文件夹里面的除了入口 exe 之外的其他文件

以上的 PublishFolderCleaner 工具的作用就是将发布文件夹里面的所有文件，除了入口 exe 之外的文件，都放入到 lib 文件夹里面，然后修改入口 exe 文件的逻辑，让入口 exe 可以从 lib 文件夹里面读取入口 dll 文件，从而实现此功能

## 例子

我创建了一个基于 .NET 5 的 WPF 应用，给此应用加上 [dotnetCampus.PublishFolderCleaner](https://www.nuget.org/packages/dotnetCampus.PublishFolderCleaner) 的 NuGet 包

接着使用命令行进行发布，发布命令如下

```
dotnet publish -r win-x64 -c release --self-contained
```

接着进入到 `bin\Release\net5.0-windows\win-x64\publish\` 文件夹，可以看到此文件夹只有存放一个 exe 和一个 lib 文件夹，如下

```
|   WhihuqeabaLeelurlallball.exe
|   
\---lib
    |   clrcompression.dll
    |   clretwrc.dll
    |   clrjit.dll
    |   coreclr.dll
    |   createdump.exe
    |   WhihuqeabaLeelurlallball.deps.json
    |   WhihuqeabaLeelurlallball.dll
    |   WhihuqeabaLeelurlallball.pdb
    |   WhihuqeabaLeelurlallball.runtimeconfig.json
    |   WindowsBase.dll
    |   WindowsFormsIntegration.dll
    |   wpfgfx_cor3.dll
    |   // 忽略很多文件
    +---zh-Hans
    |       Microsoft.VisualBasic.Forms.resources.dll
    |       PresentationCore.resources.dll
    |       // 忽略很多文件
    |       
    \---zh-Hant
            Microsoft.VisualBasic.Forms.resources.dll
            // 忽略很多文件
```

## 代码

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/24c0c22f4a0bb292893ac09aba2f14b3b84a2d6e/WhihuqeabaLeelurlallball) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/24c0c22f4a0bb292893ac09aba2f14b3b84a2d6e/WhihuqeabaLeelurlallball) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 24c0c22f4a0bb292893ac09aba2f14b3b84a2d6e
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 WhihuqeabaLeelurlallball 文件夹

可以通过这个简单的例子试试效果

## 原理

本文使用的 [PublishFolderCleaner](https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK) 工具，在 [GitHub](https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK) 上完全开源，属于我所在团队构建工具链的工具，请看 [https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK](https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK)

核心机制就是添加构建调度步骤，在发布之后执行移动文件和修改入口 exe 两个步骤

其中添加构建调度的逻辑代码如下

```xml
<Project>
  <Target Name="MoveThePublishFolderToLibFolder" AfterTargets="Publish">

    <PropertyGroup>
      <PublishFolderCleanerCommandArgs>dotnet "$(MSBuildThisFileDirectory)..\tools\net5.0\PublishFolderCleaner.dll" -p "$(PublishDir) " -a "$(AssemblyName)"</PublishFolderCleanerCommandArgs>
    </PropertyGroup>

    <Exec Command="$(PublishFolderCleanerCommandArgs)"></Exec>
  </Target>
</Project>
```

也就是在发布完成之后，通过 dotnet 命令调用 PublishFolderCleaner 工具，如上面代码可以看到这是一个 .NET 5 的工具，要求当前开发者的开发环境里面安装有 .NET 5 才能执行此工具

在 PublishFolderCleaner 工具里面完成如上两个步骤，将原有的放在发布文件夹里面的文件全部放入到里层的 lib 文件夹，再通过修改入口 exe 可执行文件，也就是 AppHost 文件，让入口 exe 从原本的相同文件夹读取入口 dll 替换为从 lib 文件夹里面读取入口 dll 文件

关于修改 AppHost 文件的知识，请参阅 [dotnet core 应用是如何跑起来的 通过AppHost理解运行过程](https://blog.lindexi.com/post/dotnet-core-%E5%BA%94%E7%94%A8%E6%98%AF%E5%A6%82%E4%BD%95%E8%B7%91%E8%B5%B7%E6%9D%A5%E7%9A%84-%E9%80%9A%E8%BF%87AppHost%E7%90%86%E8%A7%A3%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.html ) 和 [dotnet 桌面端基于 AppHost 的配置式自动切换更新后的应用程序路径](https://blog.lindexi.com/post/dotnet-%E6%A1%8C%E9%9D%A2%E7%AB%AF%E5%9F%BA%E4%BA%8E-AppHost-%E7%9A%84%E9%85%8D%E7%BD%AE%E5%BC%8F%E8%87%AA%E5%8A%A8%E5%88%87%E6%8D%A2%E6%9B%B4%E6%96%B0%E5%90%8E%E7%9A%84%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E8%B7%AF%E5%BE%84.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
