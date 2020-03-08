# git 上传当前分支

因为我现在的分支是的名很长，每次需要上次当前分支需要写很多代码，是不是有很简单方法上传当前分支。

<!--more-->
<!-- CreateTime:2019/8/31 16:55:59 -->

<!-- 标签：git -->

如果要上传一个分支到仓库 origin 那么就需要使用下面的命令

```csharp
  git push origin 分支
```

我存在下面几个分支

```csharp
 t/lindexi/Avalonial_Grid_Arrange
 t/lindexi/Avalonial_Grid_Infinity
```

我在上传的时候需要写很多代码，至少很难用 tab 出我现在的分支

我在网上找到一个方法，用来上传当前的分支

```csharp
  git push origin HEAD
```

但是我发现每次这样写还是很长，再告诉大家一个方法

```csharp
 git config --global push.default current
```

设置默认使用`git push `就是上传当前分支

可以设置 push.default 的值为

 - nothing  不上传任何分支
 - matching 上传所有分支
 - upstream/tracking  上传当前跟踪分支
 - current  上传当前分支

 实际上还有更多，请看[Git - git-config Documentation](https://git-scm.com/docs/git-config.html#git-config-pushdefault )

https://stackoverflow.com/q/14031970/6116637

![](https://i.loli.net/2018/05/19/5affbfbc1926d.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
