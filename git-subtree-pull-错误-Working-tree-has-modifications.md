
# git subtree pull 错误 Working tree has modifications

git subtree 是不错的东西，用于 git 管理子项目。
本文记录我遇到问题和翻译[网上](https://stackoverflow.com/a/18608538/6116637)的答案。

<!--more-->



当我开始 pull 的时候，使用下面的代码

```csharp
git subtree pull --prefix=<本地子项目目录> <远程库仓库地址 | 远程库别名> <分支> --squash
```

其中`--squash` 参数是把子项目的记录合成一次 commit 提交到主项目，这样主项目只是合并一次 commit 记录。

但是在我执行这句代码的时候，出现下面的错误

```csharp
Working tree has modifications.  Cannot add.
```

当我检查本地是否有没提交的保存时候，没有找到

```csharp
git status
```

这个问题是因为`git diff-index HEAD`返回结果，即使本地没提交，解决这个问题很简单。切换到本地另一个分支然后切换回来，这样就可以解决

```csharp
git checkout 其他分支
git checkout master
```

如果执行了上面的命令还无法使用，请告诉我。

[Git subtree 管理子项目包使用小结](https://blog.zthxxx.me/posts/Git-subtree-Manage-Subpackages-Usage-Summary/)

https://stackoverflow.com/a/18608538/6116637





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。