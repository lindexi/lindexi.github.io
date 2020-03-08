# C# 已知点和向量，求距离的点

已知一个点 P 和向量 v ，求在这个点P按照向量 v 运行距离 d 的点 B 。

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


<!-- math -->

已经知道了一个点 P 和他运动方向 v ，就可以通过这个求出距离点 P 为 d 的点 B。


![](http://image.acmx.xyz/lindexi%2F2018581454142946.jpg)

首先把 v 规范化，规范化的意识是向量的摸变为1

![](http://image.acmx.xyz/lindexi%2F20185101542383756.jpg)

画一张图来就是把图片灰色向量修改为黑色向量

![](http://image.acmx.xyz/lindexi%2F20185101542523183.jpg)

那么 B 的计算可以转换为求 B 的向量

![](http://image.acmx.xyz/lindexi%2F2018510154558411.jpg)

这时的 B 向量可以使用下面的公式

![](http://image.acmx.xyz/lindexi%2F2018510154712864.jpg)

因为 B 的坐标和 B 向量是相同，所以 B 的坐标就是

$$
B=(A_x,A_y)+(L·V'_x,L·V'_y) \\
 =(A_x+L·V'_x,A_y+L·V'_y)
$$



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
