# git 分支改名

给一个git分支改名的方法很简单

<!--more-->
<!-- CreateTime:2018/11/3 12:49:09 -->


<div id="toc"></div>

如果对于分支不是当前分支，可以使用下面代码：

```csharp
    git branch -m 原名 新
```

例如当前的分支是 master 分支，想要修改 t/lindexi 分支为 t/lindexiIsDoubi

可以使用下面的代码

```csharp
git branch -m t/lindexi t/lindexiIsDoubi
```

如果是修改当前的分支，直接修改为新的名字就可以了

```csharp
        git branch -m 新的分支名 
```

如当前的分支是 master 分支，想要将他修改 t/lindexiIsDoubi 分支，可以使用下面的代码

```csharp
git branch -m t/lindexiIsDoubi
```

参见：

http://zengrong.net/post/1746.htm

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。