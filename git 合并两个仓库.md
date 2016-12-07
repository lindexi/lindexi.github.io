我有两个仓库，一个是gitbook在写一本UWP入门，一个是放在github的垃圾，这个是我想要开个人网站，但是做的还是不行https://github.com/lindexi/lindexi.github.io结果发现我需要做html，本来的文件没法直接转过去，但是我又不想使用第三方工具，于是最后我想着自己来写一个，于是就做了winMarkdown，win10软件，不过已经几个月没做

好了还是回到问题，我想把两个git合并

<!--more-->

首先用git bash 到我的一个仓库，作为需要合并的仓库

```
cd 仓库
```

添加我要合并仓库

```
#git remote add 仓库 仓库可以是远程仓库
git remote add lindexi git@github.com:lindexi/lindexi.github.io.git
```
从远程仓库下载，这时我们弄个新的

```
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



