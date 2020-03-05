我有两个仓库，一个是gitbook在写一本UWP入门，一个是放在github的垃圾，这个是我想要开个人网站，但是做的还是不行https://github.com/lindexi/lindexi.github.io结果发现我需要做html，本来的文件没法直接转过去，但是我又不想使用第三方工具，于是最后我想着自己来写一个，于是就做了winMarkdown，win10软件，不过已经几个月没做

好了还是回到问题，我想把两个git合并

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

首先用git bash 到我的一个仓库，作为需要合并的仓库

```csharp
cd 仓库
```

添加我要合并仓库

```csharp
# git remote add 仓库 仓库可以是远程仓库
git remote add lindexi git@github.com:lindexi/lindexi.github.io.git
```
从远程仓库下载，这时我们弄个新的

```csharp
git branch lindexi
git checkout lindexi
git fetch lindexi
git merge lindexi/master
解决冲突
git add .
git commit -m "合并"
git push lindexi lindexi:master
git checkout master
git merge lindexi
git branch -d lindexi
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。




