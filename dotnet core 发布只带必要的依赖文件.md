# dotnet core 发布只带必要的依赖文件

在使用 dotnet core 发布独立项目的时候，会带上大量依赖的库，但是通过微软提供的工具可以去掉一些在代码没有用到的库。

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


本文介绍的工具是 Microsoft.Packaging.Tools.Trimming 这是一个预览的工具，需要从 myget.org 下载

通过这个工具可以分析有哪些 dll 不是必须的

这个工具使用方法很简单，不过需要先点击 nuget 源添加 https://dotnet.myget.org/F/dotnet-core/api/v3/index.json 才可以

添加方法十分简单，本文这里就不说了

通过 nuget 安装 Microsoft.Packaging.Tools.Trimming 库，这个库是预览版，需要勾选预览才能找到

然后编辑一下 csproj 文件

```csharp
    <PropertyGroup>
        <TrimUnusedDependencies>true</TrimUnusedDependencies>
        <RootPackageReference>false</RootPackageReference>
    </PropertyGroup>
```

这里添加的属性 TrimUnusedDependencies 就是表示删除不需要的依赖，这里的依赖就是相对于 Root 寻找的依赖，也就是不需要的 PackageReferences 都不会放在发布文件

如果不知道如何编辑 csproj 文件，有一个简单的方法就是在发布的时候加上参数 `/p:TrimUnusedDependencies=true` 就可以

```csharp
dotnet publish --self-contained true /p:TrimUnusedDependencies=true
```

建议是在 build 的时候也使用 `/p:TrimUnusedDependencies=true` 参数，这样就可以发现一些动态使用的类没有引用

实际我测试通过这个方法没有去掉多少dll不如使用 CoreRT 编译为 Native 文件

[Reducing the size of self-contained .NET Core applications](https://ianqvist.blogspot.com/2018/01/reducing-size-of-self-contained-net.html?tdsourcetag=s_pctim_aiomsg )

[Creating a Minimal ASP.NET Core Windows Container](https://blogs.msdn.microsoft.com/webdev/2017/11/09/creating-a-minimal-asp-net-core-windows-container/ )

[standard/trimming.md at release/2.0.0 · dotnet/standard](https://github.com/dotnet/standard/blob/release/2.0.0/Microsoft.Packaging.Tools.Trimming/docs/trimming.md )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
