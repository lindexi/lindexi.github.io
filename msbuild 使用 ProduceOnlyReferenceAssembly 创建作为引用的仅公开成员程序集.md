# msbuild 使用 ProduceOnlyReferenceAssembly 创建作为引用的仅公开成员程序集

本文告诉大家如何使用 msbuild 的 ProduceOnlyReferenceAssembly 功能，将某个程序集里面仅导出其中的公开成员定义，而不包含具体的实现的方法

<!--more-->
<!-- CreateTime:2021/7/8 19:41:55 -->

<!-- 发布 -->

有一些 NuGet 包在发布的时候，为了做一些有趣的业务，期望只是包含程序集的公开成员定义，如公开的方法和公开的属性和枚举等，但是不要包含具体的实现逻辑代码。这样的业务会用在为了减少 NuGet 包的体积，如为了制作插件使用的 NuGet 包。或者说在特定平台上不知道如何实现，只是为了辅助构建通过而已，如我在 Unity 3D 上提供的一些库，表示我不知道如何实现，我只是为了让构建能通过而已

使用 ProduceOnlyReferenceAssembly 可以让输出的程序集 dll 或 exe 里面只是包含了公开的成员的定义，但不包含具体的实现代码。这样的程序集是仅仅作为被引用的程序集使用的，不能被实际调用

下面来告诉大家如何构建这样的程序集，构建有两个方法，第一个是放在 csproj 项目文件里面。如在项目文件里面添加如下代码

```xml
<ProduceOnlyReferenceAssembly>true</ProduceOnlyReferenceAssembly>
```

添加之后的 csproj 文件代码大概如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net45;net46;netstandard2.0;netcoreapp3.1</TargetFrameworks>
    <ProduceOnlyReferenceAssembly>true</ProduceOnlyReferenceAssembly>
  </PropertyGroup>
</Project>
```

此时执行和之前一样的构建代码，如 `msbuild` 命令，在构建完成之后输出的 dll 可以看到比之前的小很多。通过 dnspy 等工具，可以看到这个 Dll 里面的所有类的方法都没有具体的实现


但是在很多应用上，更改 csproj 加上以上代码不现实。咱可以通过在构建的时候，修改构建命令来打出仅作为引用的程序集，如执行以下代码

```
msbuild /p:ProduceOnlyReferenceAssembly=true
```

此时构建出来的 dll 就是只读程序集，里面不包含具体的实现

此构建方法适合在库里面进行，如果是在一个大的应用项目里面构建，如果发现构建不通过，就需要你了解很多构建相关的知识才能解决哈

更多请看 [C# Compiler Options - code generation options](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/code-generation#produceonlyreferenceassembly?WT.mc_id=WD-MVP-5003260 ) 

官方文档是 [Reference assemblies](https://docs.microsoft.com/en-us/dotnet/standard/assembly/reference-assemblies?WT.mc_id=WD-MVP-5003260 )

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 

如果不想从源代码生成，期望从 DLL 里面重新导出，请参阅 [dotnet 使用 Refasmer 从现有的 DLL 里面导出公开的成员组装出新的仅作为引用用途的程序集](https://lindexi.gitee.io/post/dotnet-%E4%BD%BF%E7%94%A8-Refasmer-%E4%BB%8E%E7%8E%B0%E6%9C%89%E7%9A%84-DLL-%E9%87%8C%E9%9D%A2%E5%AF%BC%E5%87%BA%E5%85%AC%E5%BC%80%E7%9A%84%E6%88%90%E5%91%98%E7%BB%84%E8%A3%85%E5%87%BA%E6%96%B0%E7%9A%84%E4%BB%85%E4%BD%9C%E4%B8%BA%E5%BC%95%E7%94%A8%E7%94%A8%E9%80%94%E7%9A%84%E7%A8%8B%E5%BA%8F%E9%9B%86.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
