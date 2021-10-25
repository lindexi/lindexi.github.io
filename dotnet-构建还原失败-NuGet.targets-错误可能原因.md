
# dotnet 构建还原失败 NuGet.targets 错误可能原因

我在一次断电关机之后，发现我所有的项目都构建不通过了，提示在 NuGet.targets 文件的第 130 行错误。原因就是存在有某个被项目引用的 NuGet 包被损坏，在进行 NuGet 还原时读取这个包出错

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

提示内容大概如下

```
C:\Program Files\dotnet\sdk\6.0.100-rc.2.21505.57\NuGet.targets(130,5): error : '.', hexadecimal value 0x00, is an invalid character. Line 1, position 1.
```

大部分此时都会加上某个项目或 sln 文件，但实际上错误内容和此无关，更多的是某个 NuGet 包被损坏

解决方法很简单，删除 `C:\Users\你的用户名\.nuget\packages\` 文件夹下对应的 NuGet 包，或者清空此文件夹都可以。这个文件夹是 NuGet 的本机缓存文件夹，删除之后将会重新去 nuget.org 等上面拉 nuget 包，因此大部分情况下是可以安全删除的





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。