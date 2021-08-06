
# WPF 基于 .NET 5 框架和 .NET 6 的 SDK 进行完全单文件发布

本文来告诉大家如何基于 .NET 5 框架和 .NET 6 SDK 进行完全单文件发布，这是对 WPF 应用程序进行独立发布，生成的是完全单文件的方法

<!--more-->



<!-- 发布 -->

在之前的版本，尽管也是基于 .NET 5 框架的 WPF 应用，然而在 .NET 5 的 SDK 下，除非是采用框架依赖的方法，否则大部分应用发布作为单文件将会运行失败。在 .NET 6 的 SDK 下，官方修复了一些文档，对于大部分 WPF 应用程序来说，可以在 .NET 6 的 SDK 下，可以发布为完全的单文件

发布方法是在参数加上 `-p:PublishSingleFile=true` 和 `-p:IncludeNativeLibrariesForSelfExtract=true` 两个参数

```
dotnet publish -r win-x86 -c release -p:PublishSingleFile=true -p:IncludeNativeLibrariesForSelfExtract=true
```

用此方法的要求是，如果在 WPF 的 XAML 或者业务逻辑里面，有用到 Content 的内容都需要进行更改，有使用到当前应用程序所在文件夹附近的其他的文件的逻辑，也需要进行更改。毕竟是单文件发布，也只有一个文件

更改的方法是将原本的读取文件的逻辑，放入到程序集里面，通过程序集读取

当前的 WPF 暂时不支持裁剪的功能，完全单文件无框架依赖发布的空应用有 130M 左右

[Single file application - .NET](https://docs.microsoft.com/en-us/dotnet/core/deploying/single-file?WT.mc_id=WD-MVP-5003260 )






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。