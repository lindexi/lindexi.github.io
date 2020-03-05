# Roslyn 理解 msbuild 的清理过程

在开发的时候，小伙伴会使用右击解决方案，点击清理解决方案。在这个按钮点击的背后 msbuild 做了什么？为什么很多时候的清理之后还存在一堆文件？如何让自己想的 Target 也支持清理

<!--more-->
<!-- CreateTime:2019/7/3 18:21:25 -->

<!-- csdn -->
<!-- 标签：Roslyn,MSBuild,编译器 -->

在点击清理按钮的时候，将会执行 msbuild 的清理代码，对应的 Clean 这个 Target 将会被执行

在 Clean 执行的时候，是如何让 msbuild 知道哪些文件可以删除，哪些文件不能被删除？在编译的过程和清理的过程是独立，如何在清理的过程能知道编译的时候生成了哪些文件是属于可删除的

两个过程之间的通信最简单的就是使用文件，在 msbuild 编译的时候，将会生成一个 `.FileListAbsolute.txt` 文件，可以在清理的时候通过读取这个文件知道有哪些内容是可以删除的

尝试新建一个项目，在这个项目的 `obj` 文件夹里面找到 `.FileListAbsolute.txt` 文件，打开这个文件的内容，尝试在里面删除或添加一项，然后执行清理命令

此时将会发现在 `.FileListAbsolute.txt` 列出的文件将会被删除，没有被列出的文件将不会删除

这也就是为什么使用清理之后还有很多文件没有删除，因为这些文件都没有加入到 `.FileListAbsolute.txt` 列表

那么使用这个机制，如何让自己写的 Target 支持在清理的时候删除创建的临时文件？

此时有两个方法，第一个方法就是使用 msbuild 清理的方法，在编译的时候添加创建的文件进入可以删除的文件列表

通过 `$(IntermediateOutputPath)$(CleanFile)` 可以拿到对应的清理文件

于是在这个文件里面写入需要删除的文件列表就可以，注意写入的是绝对路径

上面方法的好处是如果在编译的时候会创建一些随机的文件，那么在清理的过程可以找到这些随机创建的文件

另一个方法是在执行清理的时候运行自己的代码

创建一个 PropertyGroup 在里面添加自己的 target 名在 CleanDependsOn 里面，如下面代码将 Lindexi 添加到清理的依赖项

```csharp
<PropertyGroup>
    <CleanDependsOn>$(CleanDependsOn);Lindexi</CleanDependsOn>
</PropertyGroup>
```

将会在清理的时候，先执行添加的依赖项 Lindexi 的代码

```csharp
<Target Name="Lindexi">
    清理的代码
</Target>
```

[让 MSBuild Target 支持 Clean - walterlv](https://blog.walterlv.com/post/support-clean-for-msbuild-target.html )

[Microsoft.Common.CurrentVersion.targets](https://github.com/Microsoft/msbuild/blob/9354c727bd70450912c882dfeaf8941a67dc2f66/src/Tasks/Microsoft.Common.CurrentVersion.targets#L4725 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
