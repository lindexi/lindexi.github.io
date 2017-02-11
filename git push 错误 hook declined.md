# git push 错误 hook declined 

我把仓库上传到 gogs 出现错误，提示如下 `remote: hooks/update: line 2: E:/gogs/gogs.exe: No such file or directory`

gogs 仓库无法上传，一个原因是移动了gogs，如果把gogs放在移动U盘，插入时，上传经常出现这个问题。

<!--more-->

在 push 的提示：


```csharp
  git push origin master
Counting objects: 32, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (24/24), done.
Writing objects: 100% (32/32), 2.00 MiB | 0 bytes/s, done.
Total 32 (delta 19), reused 13 (delta 7)
remote: hooks/update: line 2: E:/gogs/gogs.exe: No such file or directory
remote: error: hook declined to update refs/heads/master
To http://127.0.0.1:3000/lindexi/gogs.git
 ! [remote rejected] master -> master (hook declined)
error: failed to push some refs to 'http://127.0.0.1:3000/lindexi/gogs.git'
```

那么如何解决。

可以看到是 hook 炸了，其中 `update` 文件出现找不到路径。

原因是我把 gogs 从E盘移动到D盘，于是 提交 gogs 仓库出现 `remote: hooks/update: line 2: E:/gogs/gogs.exe: No such file or directory` 。

解决方法：

打开 gogs 仓库 gogs ，注意我的gogs仓库之前所在是 `E:\gogs\lindexi\gogs.git\` 所以我移动了路径但是里面的路径不会变，打开 `update` 文件。

可以看到：


```csharp
  #!/usr/bin/env bash
  "E:/gogs/gogs.exe" update $1 $2 $3 --config='E:/gogs/custom/conf/app.ini'
```

于是我修改 E盘的路径到我现在使用的路径，就好了。

如果对于 gogs 仓库在上传时出现的问题，也可以联系我：lindexi_gd@163.com

一般有时间我会去看看。

如果对于文章有疑问，欢迎交流。
