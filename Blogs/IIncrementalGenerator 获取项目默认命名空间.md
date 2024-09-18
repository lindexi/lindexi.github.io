---
title: IIncrementalGenerator 增量 Source Generator 生成代码入门 获取项目默认命名空间
description: 本文将告诉大家如何在分析器里面获取到项目的默认命名空间
tags: Roslyn,MSBuild,编译器,SourceGenerator,生成代码
category: 
---

<!-- CreateTime:2023/8/21 19:33:32 -->
<!-- 标题： IIncrementalGenerator 获取项目默认命名空间 -->

<!-- 发布 -->
<!-- 博客 -->

<!-- 标签：Roslyn,MSBuild,编译器,SourceGenerator,生成代码 -->

在 Roslyn 分析器里面读取项目的默认命名空间，可以通过读取项目的属性配置实现。通过 [IIncrementalGenerator 增量 Source Generator 生成代码入门 读取 csproj 项目文件的属性配置](https://blog.lindexi.com/post/IIncrementalGenerator-%E5%A2%9E%E9%87%8F-Source-Generator-%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81%E5%85%A5%E9%97%A8-%E8%AF%BB%E5%8F%96-csproj-%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6%E7%9A%84%E5%B1%9E%E6%80%A7%E9%85%8D%E7%BD%AE.html ) 的方法配置读取即可

以下核心代码可以放入到安装分析器的项目的 csproj 项目文件，也可以放入到分析器所在 NuGet 包的 `XxxPackage.props` 文件里面，这里的 `XxxPackage.props` 就是对应 NuGet 包的 props 文件，详细请参阅 [Roslyn 打包自定义的文件到 NuGet 包](https://blog.lindexi.com/post/Roslyn-%E6%89%93%E5%8C%85%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%87%E4%BB%B6%E5%88%B0-NuGet-%E5%8C%85.html )

```xml
  <ItemGroup>
    <CompilerVisibleProperty Include="RootNamespace" />
  </ItemGroup>
```

在 IIncrementalGenerator 增量构建代码里面即可使用以下代码读取项目的 RootNamespace 默认命名空间

```csharp
                    if (provider.GlobalOptions.TryGetValue("build_property.RootNamespace", out var rootNamespace))
                    {
                    }
```

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/a749cb7f7866efeb4f922469394a4a71693037ea/LainewihereJerejawwerye) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/a749cb7f7866efeb4f922469394a4a71693037ea/LainewihereJerejawwerye) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin a749cb7f7866efeb4f922469394a4a71693037ea
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin a749cb7f7866efeb4f922469394a4a71693037ea
```

获取代码之后，进入 LainewihereJerejawwerye 文件夹

更多源代码生成，请看官方的 [Source Generators Cookbook](https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.cookbook.md)

更多关于我博客请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
