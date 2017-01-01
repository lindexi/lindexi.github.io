rebase可以修改记录，我总是做小更改就提交，仓库有好多看起来很乱的

git没有可以把最后一个提交提交到服务器的能力，可以用rebase

rebase可以合并提交，使用简单

<!--more-->

先使用分支做更改

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

记下那提交

把更改合并master

```csharp
git merge 更改
```

用rebase把更改多个合为最后一个

```csharp
git rebase -i 记下的提交
```

![这里写图片描述](image/20151226160007835.jpg)

在打开的文件的`pick`除了第一个pick，改为s

用i修改，先按i，修改

![这里写图片描述](image/20151226160057537.jpg)

![这里写图片描述](image/20151226160137293.jpg)

修改完，按esc，`:wq`保存

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

提交就是最后一个





