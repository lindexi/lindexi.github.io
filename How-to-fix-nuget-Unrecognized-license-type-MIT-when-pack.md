
# How to fix nuget Unrecognized license type MIT when pack

When I packaging license within the nupkg, I will using License to replace licentUrl.

<!--more-->



I using this code to set the license as MIT but it can not pack.

```
<license type="MIT"/>
```

Because it is a newest feature.

If your nuget version is 5.0.2 that you should use this code.

```
<license type="expression">MIT</license>
```

[Packaging License within the nupkg · NuGet/Home Wiki](https://github.com/NuGet/Home/wiki/Packaging-License-within-the-nupkg )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。