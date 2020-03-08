# git无法pull仓库refusing to merge unrelated histories

本文讲的是把git在最新2.9.2，合并pull两个不同的项目，出现的问题

如何去解决 `fatal: refusing to merge unrelated histories` 合并两个不同历史的仓库

<!--more-->
<!-- CreateTime:2019/11/29 8:21:50 -->


一开始是我在 [Github](https://github.com/iip-easi/EncodingNormalior) 新建一个仓库，写了 License ，然后把本地一个写了很久仓库上传。

因为已经创建了和本地不同的，不是一个空仓库，在上传之前先pull，因为两个仓库不同，发现`refusing to merge unrelated histories`，无法 pull 所以上传失败，谷歌了很久才发现为何这样。

原来在 2.9.2 之后，不可以合并不同没有相同结点的分支，如果需要合并两个不同结点的分支，那么需要在`git pull `添加一句代码`--allow-unrelated-histories`。于是修改之后的代码，假如源是origin，分支是master，那么请看下面代码。

```csharp
git pull origin master --allow-unrelated-histories
```

这个功能是可以让大家不要把仓库上传错了，如果会加上这个代码，那么就是自己确定了上传。之前很容易就把代码传错了，现在可以看到，如果上传的仓库不是之前在上传的仓库，是一个新的仓库，那么就会提示需要添加上面代码才能上传

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。