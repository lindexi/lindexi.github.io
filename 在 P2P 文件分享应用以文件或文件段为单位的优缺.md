# 在 P2P 文件分享应用以文件或文件段为单位的优缺

在我用了好几个 P2P 文件分享应用之后，我对比了多个应用对资源的处理，可以看到基本上可以分为文件夹级的、文件级的和文件里面的一段段级的。本文将会告诉大家使用不同方法的优点和可能的坑方便大家在开发时候选择

<!--more-->
<!-- CreateTime:2019/9/11 10:23:27 -->

<!-- csdn -->

<!-- 标签: P2P -->

## 存储冗余

用文件作为单位的，例如 UTorrent 工具，本身存储的文件就是用户下载的文件，除了用户下载文件之外只需要 Torrent 文件作为文件信息保存

而用文件段作为单位的，例如 IPFS 工具，将需要额外的空间存储文件段，因为一个不同的文件可能存在相同的文件段。例如 git 管理，将会多了一份文件大小

而文件段作为单位的，依然可以使用原文件作为资源，此时需要额外记录文件偏移量

![](http://image.acmx.xyz/lindexi%2F2019911101712215)

## 索引

用文件作为单位的，可以直接指定文件 id 作为索引，从而拿到文件内容

用文件段作为单位的，从文件 id 拿到了文件包含的文件段，从文件包含的文件段里面的索引拿到文件段内容

## 传输

使用文件作为单位的，需要使用下载链机制，这样才能作为支持多个设备提供下载

使用文件段作为单位的，默认就支持让一个文件从多个设备下载，因为一个文件包含多段，每一段都可以进行不影响的下载

## 校验

整个文件进行校验的速度会比较慢，而使用文件段作为校验的，可以在每一段下载完成之后就进行每一段的校验

[完整的 P2P 应用需要包含哪些功能](https://blog.lindexi.com/post/%E5%AE%8C%E6%95%B4%E7%9A%84-P2P-%E5%BA%94%E7%94%A8%E9%9C%80%E8%A6%81%E5%8C%85%E5%90%AB%E5%93%AA%E4%BA%9B%E5%8A%9F%E8%83%BD.html)

[分布式文件系统 IPFS 与 FileCoin](https://draveness.me/ipfs-filecoin )

[如何在IPFS里面上传一张图片 - omnispace的博客 - CSDN博客](https://blog.csdn.net/omnispace/article/details/79698667 )

[IPFS: BitSwap协议(数据块交换) - omnispace的博客 - CSDN博客](https://blog.csdn.net/omnispace/article/details/79698802 )

[OpenBazaar](https://openbazaar.org/ )

[Fastdfs分布式文件系统 - zhongliwen1981的专栏 - CSDN博客](https://blog.csdn.net/zhongliwen1981/article/details/100395035 )

[IPFS：建立一个静态网站 - lzl001的专栏 - CSDN博客](https://blog.csdn.net/lzl001/article/details/81904390 )

[【我的区块链之路】- 谈一谈IPFS原理及玩法 - qq_25870633的博客 - CSDN博客](https://blog.csdn.net/qq_25870633/article/details/82027510 )

[详解IPFS的本质、技术架构以及应用 - 老杨QQ122209017的博客 - CSDN博客](https://blog.csdn.net/sinat_34070003/article/details/80396198 )

[IPFS 使用入门 - 深入浅出区块链 - CSDN博客](https://blog.csdn.net/xilibi2003/article/details/85317187 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
