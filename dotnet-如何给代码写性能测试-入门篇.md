
# dotnet 如何给代码写性能测试 入门篇


<!--more-->


<!-- 草稿 -->


所测试内容的占比是多少。测试方法调用，方法里面使用 ThreadSleep 方法一秒，测试说明方法调用很慢。这是错误的。直接使用 DateTime 进行性能测试。没有分设备状态，即不同CPU以及系统忙碌情况不相同


不到一分钱、一分钱、不到一毛钱、半毛钱、不超过一毛钱、一毛钱、不到一块钱、一块钱、几块钱、不要钱、毛毛雨、一点点




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。