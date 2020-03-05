# git rebase 合并多个提交

rebase可以修改记录，我总是做小更改就提交，仓库有好多看起来很乱的

git没有可以把最后一个提交提交到服务器的能力，可以用rebase来做到把多个提交合并为一个。使用这个命令很简单，下面就来告诉大家如何使用这个命令


<!--more-->
<!-- CreateTime:2018/2/25 11:41:26 -->


<div id="toc"></div>

先使用分支做更改，如果直接在 master 分支做，可能会因为开始做不知道步骤弄错了，这样把自己写的代码不知道放哪就不好。

下面的代码可以让大家新建一个分支并且到这个分支来做把多个提交合并为一个

```csharp
git branch 更改
git checkout 更改
```

提交更改

```csharp
git commit 更改
```

然后到主分支看最新提交

```csharp
git checkout master
git log
```

![这里写图片描述](image/20151226155916257.jpg)

记下那提交的 id 然后 把更改合并master分支

```csharp
git merge 更改的id
```

用rebase把更改多个合为最后一个

```csharp
git rebase -i 记下的提交
```

![这里写图片描述](image/20151226160007835.jpg)

在打开的文件的`pick`除了第一个pick，改为s

修改方法：按下 i 修改

![这里写图片描述](image/20151226160057537.jpg)

![这里写图片描述](image/20151226160137293.jpg)

修改完，按esc，然后输入`:wq`保存

然后git会让你写修改commit，按i修改，`#`开头的是注释，commit是合并多个的。

假如我有三个提交
		

```csharp
commit : A
commit : B
commit : C

```

合并后我就可以写`commit : ABC`

![这里写图片描述](image/20151226160608688.jpg)

写完按esc，`：wq`保存

提交就是最后一个保存的 commit 这样可以多个提交合并为一个。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
