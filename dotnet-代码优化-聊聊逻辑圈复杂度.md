
# dotnet 代码优化 聊聊逻辑圈复杂度

本文属于 dotnet 代码优化系列博客
相信大家都对圈复杂度这个概念很是熟悉，本文来和大家聊聊逻辑的圈复杂度。代码优化里面，一个关注的重点在于代码的逻辑复杂度。一段代码的逻辑复杂度越高，那么维护起来的难度也就越大。衡量代码的逻辑复杂度的一个维度是通过逻辑圈复杂度进行衡量。本文将告诉大家如何判断代码的逻辑圈复杂度以及一些降低圈复杂度的套路，让大家了解如何写出更好维护的代码

<!--more-->


<!-- CreateTime:2022/11/21 8:30:16 -->


<!-- 草稿 -->

回顾以下代码设计的目标，其中一个很重要的点就是解决复杂的代码逻辑，和人类有限的智商的矛盾。假设人类的智商非常的高，无论再复杂的代码逻辑都能理解，且人类写出的逻辑也不存在漏洞，那其实很多代码设计都是不需要的。现实刚好不是，一个稍微复杂的项目，就已经不是





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。