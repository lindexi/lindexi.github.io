
# dotnet 通过 DockerfileContext 解决项目放在里层文件夹导致 VisualStudio 构建失败

本文告诉大家，如何解决 csproj 项目文件放入到里层的文件夹，不放在 sln 所在文件夹的第一层子文件夹，导致 VisualStudio 2022 在构建 docker 映像提示找不到文件的问题

<!--more-->


<!-- CreateTime:2021/12/9 20:26:04 -->

<!-- 发布 -->

在 VisualStudio 里面，可以右击 docker 文件，进行生成映像。这是默认需要此 csproj 项目文件放入到 sln 所在文件夹的第一层子文件夹里面，而如果有一些定制化的需求，放入到其他的文件夹（依然在 sln 所在的文件夹的子文件夹里面）那就需要设置 DockerfileContext 属性，告诉 Visual Studio 生成时的 Docker 映像时使用的默认上下文

如下面文件结构

```csharp
|
|  Foo.sln
|-----A
      |-----B
            |-----B.csproj
```

此时就需要在 B.csproj 上放一个 DockerfileContext 属性，此属性的作用是生成 Docker 映像时使用的默认上下文，作为相对于 Dockerfile 的路径

```xml
  <PropertyGroup>
    <DockerfileContext>..\..</DockerfileContext>
  </PropertyGroup>
```

通过此属性，才能让生成的路径是从 sln 文件开始，也就是默认生成的值。一个推荐的做法是自己删除 Dockerfile 文件，重新在 VisualStudio 里面右击添加

更多 docker 相关属性，请看 [Visual Studio 容器工具生成属性 - Visual Studio (Windows) Microsoft Docs](https://docs.microsoft.com/zh-cn/visualstudio/containers/container-msbuild-properties?view=vs-2022&WT.mc_id=WD-MVP-5003260 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。