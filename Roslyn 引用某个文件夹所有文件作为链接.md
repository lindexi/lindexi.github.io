# Roslyn 引用某个文件夹所有文件作为链接

在 SDK 格式的项目文件可以通过简单代码引用某个文件夹里面指定后缀的文件作为项目文件

<!--more-->
<!-- CreateTime:4/15/2020 8:23:09 AM -->

<!-- 发布 -->

例如我想要引用相对于 csproj 的上一层文件夹里面的 doubi 文件夹里面的所有 cs 文件，作为链接引用的方法，可以使用下面代码

```
  <ItemGroup>
    <Compile Include="..\doubi\*.cs" Link="lindexi.blog.csdn.net\%(FileName)%(Extension)" />
  </ItemGroup>
```

上面代码用到了 `FileName` 可以替换每一项的文件名，加上 Extension 就能表示路径

按照上面的代码，其实后缀名也是根据引用的文件自动添加，那么如何添加某个文件夹里面的所有 png 文件

差不多的逻辑也就能写出添加某个项目里面的所有 png 文件

```
  <ItemGroup>
    <Content Include="..\Tool.UWP\Assets\*.png" Link="Assets\%(FileName)%(Extension)" />
  </ItemGroup>
```

如果想要引用文件夹里面的所有内容，即使这些内容在文件夹的文件夹里面，也就是不在顶层文件夹，可以这样写

```
  <ItemGroup>
    <Content Include="..\Tool.UWP\Assets\**\*.png" Link="Assets\%(FileName)%(Extension)" />
  </ItemGroup>
```

可以看到上面代码添加了 `**\*.png` 其中的 `**` 表示任意一层文件夹

而有时候包含了 obj 等不期望加入的文件，可以通过 Exclude 输入，请看以下示例

```xml
Exclude="..\Tool.UWP\obj\**\*;"
```

大概的代码如下

```
  <ItemGroup>
    <Content Include="..\Tool.UWP\Assets\**\*.png" Exclude="..\Tool.UWP\obj\**\*;..\Tool.UWP\Foo\**\*" />
  </ItemGroup>
```

通过这个方法可以将原本一个大的项目，才分为多个小的项目，每个小项目独立，但是最终打包的项目将通过此 方式引用所有的小项目。同时打包的时候不仅主项目会打包，每个小的项目都可以独立打包，这样做的优势是可以提升每个小项目的内聚和降低项目之间的耦合

我现在将很多基础的工具库都用此方式管理，我将一个工具库拆分为很多个小的工具库，每个工具库只包含很小的功能，但是同时我也创建一个主项目，这个主项目里面只放很少的代码，主要代码都是通过本文的方式引用所有小工具库的代码，这样打包出来的主项目是包含全部功能的。但是这个主项目的大小居然有3M这么大了，里面全部都是代码，很多项目里面根本不需要用到这么大的工具项目，于是这些项目就可以使用小的工具库

如果此时我需要生成不同的平台的库呢？例如某个工具库我只是支持 .NET Framework 的，某个工具库我只支持 WPF 的

此时通过宏定义的方式就可以让合并到一起的主项目按照输出的不同的 NuGet 库添加不同的代码

另外我推荐使用 SourceYard 的方式制作源代码包，这样每个小的工具库被引用的时候是通过源代码被引用，这样就不会添加额外的引用文件

添加额外的引用文件将会降低软件的启动性能，详细测试请看 [C# 程序集数量对软件启动性能的影响](https://blog.lindexi.com/post/C-%E7%A8%8B%E5%BA%8F%E9%9B%86%E6%95%B0%E9%87%8F%E5%AF%B9%E8%BD%AF%E4%BB%B6%E5%90%AF%E5%8A%A8%E6%80%A7%E8%83%BD%E7%9A%84%E5%BD%B1%E5%93%8D.html)

如何使用 SourceYard 做源代码包请看 [SoureYard 官方开源项目](https://github.com/dotnet-campus/SourceYard/)

更多编译相关请看[手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html )

其实刚才上面代码写的没有保存路径的值，如何在路径使用原有项的路径请使用 `%(RecursiveDir)` 属性，更多请看 [项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://blog.walterlv.com/post/known-properties-in-csproj.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
