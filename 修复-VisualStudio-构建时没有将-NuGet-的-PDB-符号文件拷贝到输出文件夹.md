
# 修复 VisualStudio 构建时没有将 NuGet 的 PDB 符号文件拷贝到输出文件夹

本文告诉大家如何修复 VisualStudio 构建时没有将 NuGet 的 PDB 符号文件拷贝到输出文件夹的问题。如果 VisualStudio 构建时没有将 NuGet 的 PDB 符号文件拷贝到输出文件夹，那将会在调试的时候，由于找不到 PDB 符号文件而加载符号失败

<!--more-->


<!-- 发布 -->
<!-- 博客 -->
<!-- 标签： NuGet，VisualStudio，构建 -->

尽管这个坑从 2017 到现在，来来回回修了好多次，有某些时候能拷贝，有某些时候就只认 symbol 的 NuGet 包，有时候无论什么包都不认。本文将告诉大家如何强行设置拷贝 PDB 符号文件

方法是在自己的项目的 csproj 项目文件夹里面添加如下代码

```xml
<Target Name="IncludeSymbolFromReferences"
        AfterTargets="ResolveAssemblyReferences"
        Condition="@(ReferenceCopyLocalPaths) != ''">
  <ItemGroup>
    <ReferenceCopyLocalPaths
            Include="%(ReferenceCopyLocalPaths.RelativeDir)%(ReferenceCopyLocalPaths.Filename).pdb"
            DestinationSubDirectory="%(ReferenceCopyLocalPaths.DestinationSubDirectory)" />
    <ReferenceCopyLocalPaths Remove="@(ReferenceCopyLocalPaths)"
                             Condition="!Exists('%(FullPath)')" />
  </ItemGroup>
</Target>
```

以上代码表示在 ResolveAssemblyReferences 的时候，执行 IncludeSymbolFromReferences 任务，这个任务里面，将会尝试去找所有的引用的 `pdb` 文件，如果找到了，就放入到输出拷贝里面

如此即可在构建时，将引用的 NuGet 包的 DLL 对应 PDB 文件拷贝到输出文件夹，而不需要关注具体的框架版本

当然，在每个项目都拷贝以上的代码也不是好主意。以上的代码被 [SimonCropp](https://github.com/SimonCropp) 大佬封装到了 [https://github.com/SimonCropp/Cymbal](https://github.com/SimonCropp/Cymbal) 仓库里面，作为 NuGet 包发布，只需要通过 NuGet 管理器安装 `Cymbal` 即可





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。