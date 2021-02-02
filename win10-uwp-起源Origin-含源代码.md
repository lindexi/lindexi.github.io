
# win10 uwp 起源Origin 含源代码

这是一个自动机，在1940年提出，也是我们世界的Bug。
我们有三个简单规则，于是做出一个环境，他会出现很多奇怪的图形。我们发现了一百多个规则，但是我们不知道是不是只有这一百多个，最后他能推出多少个？

<!--more-->


<!-- CreateTime:2018/8/10 19:16:51 -->


<div id="toc"></div>
<!-- csdn -->

<!-- 草稿 -->

我们来说下我们的游戏，游戏每个方格就是一个人，他有8个邻居。方格的人不是存活就是死亡。

![](http://image.acmx.xyz/2128cc96-e1ca-4bdd-8812-1c2a302fa87e20161230214413.jpg)

说的方格就是中间的点。

我们定义三个规则：

 - 方格的周围有2个或三个邻居存活，那么他可以存活。

 - 如果他周围邻居小于两个存活，他因太孤单死去。如果他周围邻居大于3个存活，他因为太拥挤死去。

 - 如果他本身死去，周围有三个邻居存活，他在下回合会自动存活。

那么我们可以看到我们三个规则可以是这样：

产生的图形有：

![](http://image.acmx.xyz/f3df3fa9-4243-40b9-9862-fc48042b0c5bsimulate2.gif)

![](http://image.acmx.xyz/f3df3fa9-4243-40b9-9862-fc48042b0c5bsimulate2.gif)

当然我没有把所有的图形画出，这是静态图形，还有动态：



![](http://image.acmx.xyz/f3df3fa9-4243-40b9-9862-fc48042b0c5bsimulate1.gif)

![](http://image.acmx.xyz/f3df3fa9-4243-40b9-9862-fc48042b0c5bsimulate2.gif)

我们点一下会出现好看的花

![](http://image.acmx.xyz/f3df3fa9-4243-40b9-9862-fc48042b0c5bsimulate3.gif)

这个游戏看起来不是那么好玩

可是我们做的是去研究它，所以不好玩也没关系。本文开始说道我们世界有两个bug，一个是递归，我们来说下。

假如上帝是万能，我们递归，上帝做出上帝吃不下的饭。

于是上帝做出了上帝吃不下的饭，调用，上帝不存在上帝吃不下的饭。返回上帝做不出上帝吃不下的饭。于是上帝不是万能的。




本文的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/2971742d7fd6a140d5c4423bd4455ca65eb42f0e/DerewewebaGineyinereburha) 欢迎小伙伴访问




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。