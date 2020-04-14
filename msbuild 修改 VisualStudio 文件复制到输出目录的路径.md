# msbuild 修改 VisualStudio 文件复制到输出目录的路径

在默认的 VisualStudio 可以右击任意的文件，让这个文件在编译时复制到输出目录，但是这个选项将会在复制到输出目录时带上这个文件所在 VisualStudio 的文件夹结构。本文告诉大家几个方法让 VisualStudio 的文件可以在编译时输出到自定义的任意路径

<!--more-->
<!-- CreateTime:2020/1/19 18:08:07 -->

<!-- 发布 -->

## 文件夹到文件夹

将整个文件夹内容输出到自定义文件夹可以使用写一个 Target 的方法

如我需要将 VisualStudio 的 `dll` 文件夹的内容全部输出到输出目录，而不是输出到 输出目录下的 Dll 文件夹，可以使用下面代码

```xml
    <Target Name="CopyReferenceDll" AfterTargets="Build">
        <ItemGroup>
            <ReferenceDll Include="Dll\*.dll">
            </ReferenceDll>
        </ItemGroup>
        <Copy SourceFiles="@(ReferenceDll)" DestinationFolder="$(OutputPath)" SkipUnchangedFiles="True"></Copy>
    </Target>
```

将上面代码复制到 csproj 文件里面，放在 `</Project>` 之前就可以做到将 VisualStudio 里面的 Dll 文件夹的 `dll` 文件复制到输出文件夹

至于需要输出其他文件到其他文件夹的，看上面的代码修改就可以了，如果不知道如何修改请联系我

## 用 Link 修改路径

通过修改文件链接的地址，这个方法只能输出到输出文件夹内某个文件夹下

```xml
    <ItemGroup>
      <None Update="Dll\*.dll" Link="%(Filename)%(Extension)">
        <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
      </None>
    </ItemGroup>
```

此时的 Dll 文件夹的 dll 文件，将会重新链接到 `*.dll` 路径而不是 `Dll\*.dll` 但是此时会在 VisualStudio 看到项目的根文件夹存在很多 dll 文件

如果不想看到这些文件通过 `Visible` 属性

```xml
    <ItemGroup>
      <None Update="Dll\*.dll" Link="%(Filename)%(Extension)" Visible="false">
        <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
      </None>
    </ItemGroup>
```

## 通过 ContentWithTargetPath 项

将原本的 None 替换为 ContentWithTargetPath 项，这个项提供 TargetPath 属性，设置输出的文件

```xml
    <ItemGroup>
        <ContentWithTargetPath Include="Dll\*.dll">
            <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
            <TargetPath>%(Filename)%(Extension)</TargetPath>
        </ContentWithTargetPath>
    </ItemGroup>
```

这个方法底层也是调用了 Copy 命令

适合文件到文件的方式，将 `%(Filename)%(Extension)` 替换为具体文件，如将某个文件复制到指定的路径，这样写清真很多，请看代码

```xml
    <ItemGroup>
        <ContentWithTargetPath Include="Dll\lindexi.dll">
            <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
            <TargetPath>lindexi.dll</TargetPath>
        </ContentWithTargetPath>
    </ItemGroup>
```

本文用到了 `%(Filename)` 以及 `Copy` 和 `Target` 等，这些可以在我博客看到如何使用

[Roslyn 如何使用 MSBuild Copy 复制文件](https://blog.lindexi.com/post/Roslyn-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-MSBuild-Copy-%E5%A4%8D%E5%88%B6%E6%96%87%E4%BB%B6.html.html )

[c# - Visual Studio: How to "Copy to Output Directory" without copying the folder structure? - Stack Overflow](https://stackoverflow.com/questions/18743907/visual-studio-how-to-copy-to-output-directory-without-copying-the-folder-stru )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
