# Roslyn 在 NuGet 包中放注释 xml 文件的方法

本文告诉大家如何在打出的 NuGet 包含代码的注释，这样安装了 NuGet 的小伙伴就可以在 VS 上看到对应的方法和类的注释

<!--more-->
<!-- CreateTime:2020/7/27 9:40:40 -->


<!-- 标签：Roslyn,MSBuild,编译器 -->

在使用 SDK Style 格式，可以使用下面一句话在输出的时候添加 xml 注释文件，在打包 NuGet 添加 xml 注释

```xml
<PropertyGroup>
      <GenerateDocumentationFile>true</GenerateDocumentationFile>
</PropertyGroup>
```

上面代码在 csproj 中添加

另一个方法是指定 DocumentationFile 的路径

```xml
  <PropertyGroup>
    <DocumentationFile>bin\$(Configuration)\$(TargetFramework)\$(AssemblyName).xml</DocumentationFile>
  </PropertyGroup>
```

当然，上面这个方法需要指定路径

在 NuGet 包里面，按照规则，在对应的 xx.dll 或 xx.exe 存在对应的 xx.xml 文件，那么这个 xx.xml 文件将会被作为库的注释文件被 VS 使用

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
