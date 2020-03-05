# visualStudio 无法登陆

如果遇到 visualStudio 无法登陆，可以看下我的方法，可能有用。

<!--more-->
<!-- CreateTime:2018/11/19 15:24:15 -->


尝试关闭代理

打开设置、网络、代理，关了它，试试

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017616162926.jpg)


如果遇到下面的问题：

我们无法刷新此账户的凭据

No home tenant info found.


![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017616162719.jpg)

那么可以尝试下面的方法：


如果有人用 VS 登 AzureCN 的账户导致 VS 无法登陆MS账户，可以删除`C:\Users\【username】\AppData\Local\.IdentityService`文件里所有内容以还原VS账户设置

感谢：[王嘉祥](http://wangjiaxiang.net/) 提供的方法

参见：[VS2017 直接使用账户登录 Azure](https://www.azure.cn/documentation/articles/aog-portal-management-qa-vs2017-login/)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
