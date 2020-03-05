# 如何删除错误提交的 git 大文件

早上小伙伴告诉我，他无法拉下代码，我没有在意。在我开始写代码的时候，发现我的 C 盘炸了。因为我的磁盘是苏菲只有 256G 放了代码就没空间了，于是我查找到了原来是我的代码占用了居然有 2000+M ，寻找了很久才发现，原来我小伙伴[JAKE](http://niuyanjie.oschina.io/blog/)传了一个压缩包上去，一个1G的包。

那么如何把这个压缩包彻底从 git 删除？

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


如果需要删除 git 的历史记录，使用方法很简单，请看 [Git如何永久删除文件(包括历史记录) - shines77 - 博客园](http://www.cnblogs.com/shines77/p/3460274.html ) 。当然这个方法需要很长时间，因为提交大文件的时间不长，所以可以使用walterlv的方法 [彻底删除 Git 仓库中的文件避免占用大量磁盘空间 - walterlv](https://walterlv.oschina.io/git/2017/09/18/delete-a-file-from-whole-git-history.html )

于是远程仓库删好了，但是本地仓库还是有小伙伴拉下来，于是如何让本地的小伙伴可以使用最新的仓库？

代码就是

```csharp
git fetch -f -p
git checkout dev
git reset origin/dev --hard
git reflog expire --expire=now --all
git gc --prune=now
```

第一句代码`git fetch -f -p`的作用就是从本地拿到远程最新分支，覆盖本地存放的远程分支

第二句实际上因为主要开发分支就是 dev 分支，小伙伴就是把大文件合并到这个分支，所以需要切换到这个分支。如果你不小心把大文件提交其它的分支，记得切换的就是你提交的分支。

如果提交大文件只是在自己的分支，并且放到了远程分支，那么合并到远程开发分支，那么只需要删除自己远程分支就好了，不需要继续往下做。

第三句`git reset origin/dev --hard`是把自己的本地 dev 分支覆盖，使用远程的分支，如果开发分支不是 dev ，那么请用其他的分支。

接下来是如果在开发之前已经拉下存在大文件的开发分支并且在他之后有提交，那么需要创建一个新的分支在合并之前。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017919113234.jpg)

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201791911336.jpg)

然后把提交 pick 到新分支

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017919113440.jpg)

删除原来分支，这样就好

最后的命令是使用 gc 清掉这个提交

这时候查看自己的git 文件夹，如果文件夹还是那么大，那么说明还有一个分支是引用提交大文件，需要自己去看一下是哪个分支。

这个命令需要所有小伙伴执行，不然有一个小伙伴提交了包含大文件的提交，那么刚才做的就是白做了。

需要说明，git 如果提交一个文件，然后删除他，继续提交，那么这个文件是存在 git 中，需要使用特殊的命令才可以删除。

感谢[walterlv](https://walterlv.oschina.io/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。