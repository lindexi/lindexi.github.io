
# 在 UOS 统信安装 dotnet sdk 失败 提示 failed the verification

在 UOS 统信安装 dotnet sdk 失败 提示 You cannot install '/home/lindexi/packages-microsoft-prod.deb' that failed the verification, please go to Security Center - Security Tools - Application Security to adjust

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

这是群里的伙伴报告的问题，从错误提示看需要在安全工具里面配置允许任意应用安装。这是因为安装的 packages-microsoft-prod.deb 没有带 UOS 的签名，开启配置允许任意应用安装的方法如下

先开启开发者模式

<!-- ![](image/在 UOS 统信安装 dotnet sdk 失败 提示 failed the verification/在 UOS 统信安装 dotnet sdk 失败 提示 failed the verification0.png) -->
![](http://image.acmx.xyz/lindexi%2F20244281530303325.jpg)

再点击 “如需安装和运行未签名应用，前往安全中心进行设置” 里面的“安全中心”超链接，即可进入下面界面

<!-- ![](image/在 UOS 统信安装 dotnet sdk 失败 提示 failed the verification/在 UOS 统信安装 dotnet sdk 失败 提示 failed the verification1.png) -->
![](http://image.acmx.xyz/lindexi%2F20244281531267992.jpg)

点击切换到允许任意应用即可

相关的可能错误是 ca-certificates 导致的问题，解决方法请参阅  [修复 Debian 安装 dotnet 失败 depends on ca-certificates](https://blog.lindexi.com/post/%E4%BF%AE%E5%A4%8D-Debian-%E5%AE%89%E8%A3%85-dotnet-%E5%A4%B1%E8%B4%A5-depends-on-ca-certificates.html )

如提示 没有通过系统安全验证无法运行，请参阅 [在 UOS 统信运行 dotnet 程序提示没有通过系统安全验证无法运行](https://blog.lindexi.com/post/%E5%9C%A8-UOS-%E7%BB%9F%E4%BF%A1%E8%BF%90%E8%A1%8C-dotnet-%E7%A8%8B%E5%BA%8F%E6%8F%90%E7%A4%BA%E6%B2%A1%E6%9C%89%E9%80%9A%E8%BF%87%E7%B3%BB%E7%BB%9F%E5%AE%89%E5%85%A8%E9%AA%8C%E8%AF%81%E6%97%A0%E6%B3%95%E8%BF%90%E8%A1%8C.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。