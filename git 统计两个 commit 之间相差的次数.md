# git 统计两个 commit 之间相差的次数

本文告诉大家在一个连续的 commit 树中统计两个 commit 之间的差异的 commit 数量，也就是存在 A commit 存在而 B commit 不存在的 commit 的数量

<!--more-->
<!-- 发布 -->

可以使用下面代码统计两个 commit 或分支之间的差异的次数

```csharp
git log --oneline A ^B | 
```

这里的 A 和 B 可以替换为分支或 commit 号，如 `origin/dev` 等，下面代码统计的是 `19ef3265` 和远端的 dev 的差异数量

```csharp
git log --oneline origin/dev ^19ef3265 | wc -l
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
