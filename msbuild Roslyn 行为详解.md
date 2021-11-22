# msbuild Roslyn 行为详解

本文来告诉大家 msbuild Roslyn 的行为，本文非新手友好

<!--more-->
<!-- CreateTime:2021/11/19 19:15:23 -->

## 常用参数

[项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://blog.walterlv.com/post/known-properties-in-csproj.html )

[项目文件中的已知 NuGet 属性（使用这些属性，创建 NuGet 包就可以不需要 nuspec 文件啦） - walterlv](https://blog.walterlv.com/post/known-nuget-properties-in-csproj )

## 常用判断

[msbuild 项目文件常用判断条件](https://blog.lindexi.com/post/msbuild-%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6%E5%B8%B8%E7%94%A8%E5%88%A4%E6%96%AD%E6%9D%A1%E4%BB%B6.html )

## 多框架相关

### 调用次数

如有 Target 设置 `AfterTargets="Build"` 将在多框架下被分别调用，每个框架都会调用一次，最后还会再调用一次表示总的构建。调用次数等于框架数量加一

如在框架为 `<TargetFrameworks>net45;netcoreapp3.1;net6.0</TargetFrameworks>` 那将会分别在 `net45` `netcoreapp3.1` `net6.0` 调用一次，以及最终构建完成调用一次

### 多框架的 IntermediateOutputPath 属性值

默认是使用 IntermediateOutputPath 表示 `obj` 下的缓存文件夹，可以用来输出构建相关的缓存文件，在多框架下，默认是加上框架的路径，如 `obj\Debug\net45\` 和 `obj\Debug\net5.0\` 文件夹

随着调用的次数，各个框架构建的时候，将会带上框架的路径。在最终构建，也就是总的框架构建，调用时的值是不带上具体的框架的，如 `obj\Debug\` 文件夹

测试逻辑如下

```xml
  <Target Name="GallikufawhaGebalule" AfterTargets="Build">
    <Warning Text="IntermediateOutputPath: $(IntermediateOutputPath)" />
  </Target>
```

在多框架 `<TargetFrameworks>net45;net5.0</TargetFrameworks>` 下，以上代码输出如下

```
1>C:\lindexi\Code\Foo.csproj(17,3): warning : IntermediateOutputPath: obj\Debug\net45\
1>已完成生成项目“Foo.csproj”的操作。
1>C:\lindexi\Code\Foo.csproj(17,3): warning : IntermediateOutputPath: obj\Debug\net5.0\
1>已完成生成项目“Foo.csproj”的操作。
1>C:\lindexi\Code\Foo.csproj(17,3): warning : IntermediateOutputPath: obj\Debug\
```


## NuGet 相关

### 多框架的 BuildMultiTargeting 和 Build 文件夹下的 Target 调用次数

在 Build 文件夹下的 Target 将会在各个框架分别执行。放在 BuildMultiTargeting 的 Target 将只会执行一次，详细请看 [Roslyn 在多开发框架让 msbuild 的 Target 仅运行一次](https://blog.lindexi.com/post/Roslyn-%E5%9C%A8%E5%A4%9A%E5%BC%80%E5%8F%91%E6%A1%86%E6%9E%B6%E8%AE%A9-msbuild-%E7%9A%84-Target-%E4%BB%85%E8%BF%90%E8%A1%8C%E4%B8%80%E6%AC%A1.html )

可以同时存在 BuildMultiTargeting 和 Build 文件夹，里面的内容相互不干涉，除非有设置调用关系和引用

### 独立框架的 Target 定义属性给多框架使用

在 Build 文件夹下的 Target 将会在各个框架分别独立执行，而 BuildMultiTargeting 只会执行一次。如果在 Build 文件夹下定义属性，如下面代码

```xml
  <Target Name="BuildSourceNuGet" AfterTargets="Build">
    <PropertyGroup>
      <IntermediateOutputPathCombine>$(IntermediateOutputPathCombine);$(IntermediateOutputPath)</IntermediateOutputPathCombine>
    </PropertyGroup>

    <Warning Text="build once"/>
  </Target>
```

预期是各个框架在 `IntermediateOutputPathCombine` 属性上定义各自的 IntermediateOutputPath 路径。然而在 BuildMultiTargeting 下拿到的依然是空值

```xml
  <Target Name="BuildSourceNuGetMultiTargeting" AfterTargets="Build">
    <Warning Text="MultiTargetingBuild: $(IntermediateOutputPathCombine)"/>
  </Target>
```

因此不能在 BuildMultiTargeting 上使用到各个 Build 文件夹下的 Target 收集的属性内容

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
