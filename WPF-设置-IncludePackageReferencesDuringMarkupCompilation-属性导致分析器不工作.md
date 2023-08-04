
# WPF 设置 IncludePackageReferencesDuringMarkupCompilation 属性导致分析器不工作

本文记录在 WPF 项目里面设置 IncludePackageReferencesDuringMarkupCompilation 属性为 False 导致了项目所安装的分析器不能符合预期工作

<!--more-->


<!-- CreateTime:2023/8/2 19:37:40 -->

<!-- 发布 -->
<!-- 博客 -->

设置 IncludePackageReferencesDuringMarkupCompilation 属性为 false 将配置 WPF 在构建 XAML 过程中创建的 tmp.csproj 过程中将不引用依赖的 nuget 包。分析器默认也是通过 nuget 包方式安装的，这就导致了分析器项目没有被 tmp.csproj 项目正确使用到

如果项目里面有代码依赖分析器生成的影响语义的代码，那这部分代码将会构建不通过

详细关于 IncludePackageReferencesDuringMarkupCompilation 属性请参阅 [WPF 修复 dotnet 6 与源代码包冲突](https://blog.lindexi.com/post/WPF-%E4%BF%AE%E5%A4%8D-dotnet-6-%E4%B8%8E%E6%BA%90%E4%BB%A3%E7%A0%81%E5%8C%85%E5%86%B2%E7%AA%81.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。