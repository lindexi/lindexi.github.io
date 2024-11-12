
# 记 dotnet campus 组织为适配龙芯所做的更改

本文记录在龙芯适配过程中的 dotnet campus 开源组织的更改，这些更改仅仅只是 dotnet campus 开源组织自身的项目的更改，不涉及给任何龙芯上游的贡献

<!--more-->


<!-- CreateTime:2024/11/12 22:16:36 -->
<!-- 发布 -->
<!-- 博客 -->

## 打包构建工具

- <https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/pull/154>

修复 deb 包里面记录的架构没有包含龙芯

- <https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/pull/155>

升级更基础的库命令行调用 git 支持 Linux 龙芯版本

## 分析器版本系列

当前 2024.11 龙芯的最新 dotnet sdk 为 8.0.7 版本，这个版本分析器对应的是 4.8.0 版本。为了支持龙芯构建，降低分析器版本。详细请参阅 <https://ftp.loongnix.cn/dotnet/>

- <https://github.com/dotnet-campus/dotnetCampus.LatestCSharpFeatures/pull/5>
- <https://github.com/dotnet-campus/dotnetCampus.SourceLocalizations/pull/8>
- <https://github.com/dotnet-campus/dotnetCampus.Logger/pull/29>
- <https://github.com/dotnet-campus/dotnetCampus.Ipc/pull/156>




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。