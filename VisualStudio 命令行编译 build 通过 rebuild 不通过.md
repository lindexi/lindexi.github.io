# VisualStudio 命令行编译 build 通过 rebuild 不通过

在使用命令行编译项目，发现使用 build 可以编译通过，但是通过 rebuild 编译提示找不到项目，明明在对应的文件夹存在项目输出的 dll 文件，但是会提示找不到

<!--more-->
<!-- CreateTime:2019/12/8 11:37:51 -->

<!-- csdn -->
<!-- 发布 -->

在命令行编译的 build 和 rebuild 的不同在于使用 build 会用上次编译的内容，如果没有文件更改，那么这个项目不会重新编译。同时会从缓存拿到项目依赖顺序，获取上次编译的项目信息。而 rebuild 会先执行 clean 清理，会清理缓存重新创建项目依赖顺序

而如果 build 能通过而 rebuild 不能通过，此时请尝试清理整个项目仓库的 `bin` 和 `obj` 文件夹。在使用 git 管理的项目可以使用下面命令快速清理

```csharp
git clean -xdf
```

在执行上面命令之前请先提交本地文件

清理完成之后再次使用 build 命令，关于 msbuild 命令行请看 [MSBuild 常用参数](https://blog.lindexi.com/post/MSBuild-%E5%B8%B8%E7%94%A8%E5%8F%82%E6%95%B0.html )

如果清理完成发现通过 build 命令不通过，注意这里的不通过不是因为 nuget 没还原而没通过，那么证明现在的代码是因为依赖上次编译的内容。此时可以清理完成在 VisualStudio 打开，看错误列表内容

如果清理完成发现通过 build 命令能通过，同时用 rebuild 命令如果没有添加 /m 命令也就是并行编译没有问题，而添加了 /m 命令并行重新编译就不通过，提示某些项目的 dll 找不到，那么可能是引用项目不对

如果使用的项目是旧项目格式，不是 sdk style 格式的  csproj 文件，那么在引用项目时需要添加项目的 Guid 值。要求引用的 Guid 和项目声明的相同，和 sln 里面使用的相同

在项目文件里面，可以用记事本工具打开，可以找到 ProjectGuid 属性

```csharp
    <ProjectGuid>{7478AF9C-E871-48D0-A61E-EA5331654412}</ProjectGuid>
```

而在其他项目引用就需要这样写

```xml
    <ProjectReference Include="..\lindexi\BitStamp.csproj">
      <Project>{a4181b72-65a2-4625-a355-7dea615baa53}</Project>
      <Name>BitStamp</Name>
    </ProjectReference>
```

如上面这样写，在多线程编译会关联找不到 BitStamp 项目，原因是引用的项目的 Guid 和项目的不相同，这样就无法在编译时找到引用顺序，也就是项目没有按照顺序编译也找不到对应项目

解决方法是通过在引用右击添加引用，将项目引用取消勾选，点击确定。再右击添加引用，重新引用项目就可以

通过这一步可以在 git 等工具看到 csproj 被修改，也就是引用的 Project 属性被修改。如果这样也不成，可能是 sln 里面使用的值不对。解决方法是重新创建一个解决方案，将项目添加到解决方案，删除之前的解决方案

注意重新创建解决方案不要在原因解决方案移除项目，如果在原有解决方案移除项目将会在被移除的项目从所有引用的项目移除

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
