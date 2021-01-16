# dotnet 自动迁移 VS 2017 以前的 csproj 转为 dotnet core 的 SDK Style 风格工具

本文来安利大家一个特别好用的工具，可以自动将 VisualStudio 2017 以前版本创建的 Franken-proj 格式 的 csproj 项目文件转换为 dotnet core 的 SDK Style 风格的csproj 项目文件的工具

<!--more-->
<!-- CreateTime:2021/1/15 8:39:59 -->

<!-- 发布 -->

这是一个在 GitHub 上完全开源的仓库，请看 [https://github.com/dotnet/try-convert](https://github.com/dotnet/try-convert )

使用方法很简单，这个工具设计为 dotnet tool 工具，使用之前先使用下面命令进行安装


```
dotnet tool install -g try-convert
```

接着进入到需要转换的项目所在的文件夹

```
cd 需要转换的项目所在的文件夹
```

使用下面命令进行自动化转换

```
try-convert
```

默认能将 csproj 转换好

但是有些古老的项目比较复杂，如 WPF 的项目，此时因为有 AssemblyInfo.cs 文件需要额外处理，此时还请参阅 [解决从旧格式的 csproj 迁移到新格式的 csproj 格式 AssemblyInfo 文件值重复问题](https://blog.lindexi.com/post/%E8%A7%A3%E5%86%B3%E4%BB%8E%E6%97%A7%E6%A0%BC%E5%BC%8F%E7%9A%84-csproj-%E8%BF%81%E7%A7%BB%E5%88%B0%E6%96%B0%E6%A0%BC%E5%BC%8F%E7%9A%84-csproj-%E6%A0%BC%E5%BC%8F-AssemblyInfo-%E6%96%87%E4%BB%B6%E5%80%BC%E9%87%8D%E5%A4%8D%E9%97%AE%E9%A2%98.html)的方法解决 提示 特性重复的编译出现 CS0579 重复

```xml
    Error CS0579: “System.Reflection.AssemblyCompanyAttribute”特性重复 (1, 1)
```

更多细节请看 [从以前的项目格式迁移到 VS2017 新项目格式](https://blog.lindexi.com/post/%E4%BB%8E%E4%BB%A5%E5%89%8D%E7%9A%84%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E8%BF%81%E7%A7%BB%E5%88%B0-VS2017-%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F.html )

参考上面博客修复构建问题，或者进行手工转换项目

如果对工具使用有任何问题，欢迎到 [https://github.com/dotnet/try-convert](https://github.com/dotnet/try-convert ) 提反馈，千万不要过来问我，因为这不是我做的

更多参考博客请看

- [dotnet 新项目格式与对应框架预定义的宏](https://blog.lindexi.com/post/dotnet-%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E4%B8%8E%E5%AF%B9%E5%BA%94%E6%A1%86%E6%9E%B6%E9%A2%84%E5%AE%9A%E4%B9%89%E7%9A%84%E5%AE%8F.html)

- [让一个 csproj 项目指定多个开发框架 - walterlv](https://walterlv.github.io/post/configure-projects-to-target-multiple-platforms.html )

- [Roslyn 在 NuGet 包中放注释 xml 文件的方法](https://blog.lindexi.com/post/Roslyn-%E5%9C%A8-NuGet-%E5%8C%85%E4%B8%AD%E6%94%BE%E6%B3%A8%E9%87%8A-xml-%E6%96%87%E4%BB%B6%E7%9A%84%E6%96%B9%E6%B3%95.html)

- [将 WPF、UWP 以及其他各种类型的旧样式的 csproj 文件迁移成新样式的 csproj 文件 - walterlv](https://walterlv.github.io/post/introduce-new-style-csproj-into-net-framework.html )

- [Roslyn 禁止 sdk style csproj 默认引用 Compile 代码文件](https://blog.lindexi.com/post/Roslyn-%E7%A6%81%E6%AD%A2-sdk-style-csproj-%E9%BB%98%E8%AE%A4%E5%BC%95%E7%94%A8-Compile-%E4%BB%A3%E7%A0%81%E6%96%87%E4%BB%B6.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
