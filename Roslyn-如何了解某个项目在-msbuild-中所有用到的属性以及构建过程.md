
# Roslyn 如何了解某个项目在 msbuild 中所有用到的属性以及构建过程

使用 SDK Style 格式的 csproj 十分简化，但是实际上的构建过程需要用到超级多的逻辑，那么如何知道在 msbuild 所使用的构建过程有哪些，定义了那些属性。有那些 target 文件参与了这个项目构建
本文告诉大家一个方法，可以输出某个项目在 msbuild 中的完全使用到的 targets 和属性等

<!--more-->


<!-- 发布 -->

<!-- 标签：Roslyn,MSBuild,编译器 -->

本文需要用到 [-pp](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-command-line-reference#preprocess) 这个命令。这个命令可以输出所有内联到此项目的构建过程的文件，以及将构建过程用到的指令输出到文件中。可以通过这个命令了解构建此项目所用到的导入的文件。使用这个命令将不会构建项目

这个命令添加可以参数，参数是输出文件，如果没有参数，就输出控制台

进入某个项目，输入下面命令就可以输出在 msbuild 视角的整个构建用到的内容

```
dotnet msbuild -pp:FullProject.xml
```

输入上面命令，将会输出 `FullProject.xml` 文件

这个文件的内容很多，大概如下

<!-- ![](image/Roslyn 如何了解某个项目在 msbuild 中所有用到的属性以及构建过程/Roslyn 如何了解某个项目在 msbuild 中所有用到的属性以及构建过程0.png) -->

![](http://image.acmx.xyz/lindexi%2F2020618205568088.jpg)

可以加上平台，只输出某个平台下使用的构建过程

```
dotnet msbuild -p:TargetFramework=netcoreapp2.0 -pp:FullProject.xml
```





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。