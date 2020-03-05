
# 如何使用 dotnet 5 预览版

我说的是不是 .NET Framework 5 也不是 dotnet core 5 而是 dotnet 5 这个当前是预览版的框架

<!--more-->


<!-- 发布 -->

刚才[老司机](https://huchengv5.github.io/ )问我如何在 VisualStudio 打开[aspnetcore](https://github.com/dotnet/aspnetcore )源代码，因为这个项目用到了 dotnet 5 预览版，所以让没有安装 dotnet 5 预览版的小伙伴用不了

打开 VisualStudio 的 NuGet 包设置，添加一个源

```
https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet5/nuget/v3/index.json
```

请看下图的方法添加

![](http://image.acmx.xyz/lindexi%2F201958214432905)

如果现有在某个项目使用而不是全部使用请看 [VisualStudio 给项目添加特殊的 Nuget 的链接](https://blog.lindexi.com/post/VisualStudio-%E7%BB%99%E9%A1%B9%E7%9B%AE%E6%B7%BB%E5%8A%A0%E7%89%B9%E6%AE%8A%E7%9A%84-Nuget-%E7%9A%84%E9%93%BE%E6%8E%A5.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。