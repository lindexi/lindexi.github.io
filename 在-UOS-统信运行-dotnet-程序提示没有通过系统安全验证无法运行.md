
# 在 UOS 统信运行 dotnet 程序提示没有通过系统安全验证无法运行

本文记录 dotnet 应用程序在 UOS 统信系统上运行时，提示 没有通过系统安全验证，无法运行 的问题

<!--more-->


<!-- CreateTime:2023/9/7 9:29:42 -->

<!-- 发布 -->
<!-- 博客 -->

这个问题是因为没有开启 UOS 统信的开发者模式，直接将自己构建完成的包放上去跑导致的问题

解决方法十分简单，只需要开启开发者模式即可

控制中心 -> 通用 -> 开发者模式 -> 进入开发者模式 登录，重启

相关问题： [在 UOS 统信安装 dotnet sdk 失败 提示 failed the verification](https://blog.lindexi.com/post/%E5%9C%A8-UOS-%E7%BB%9F%E4%BF%A1%E5%AE%89%E8%A3%85-dotnet-sdk-%E5%A4%B1%E8%B4%A5-%E6%8F%90%E7%A4%BA-failed-the-verification.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。