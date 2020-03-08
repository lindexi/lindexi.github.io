# git 提交添加 emoij 文字

可能看到 git 提交是文本，就认为他无法使用表情图片，实际上 git 提交是可以添加表情。

本文告诉大家如何做出下面图片提交

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201772019141.jpg)

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


在 git 提交的时候，可以添加表情，只需要在字符串加上表示表情的文本

如需要在提交添加一个 笑 那么可以使用下面的代码

```csharp
git commit -m ":smile:"
```

可以尝试创建一个仓库，然后提交测试的代码，来看一下是不是自己的仓库支持。我尝试了 github 和 gitlab 都支持。

首先创建一个文件，然后使用下面的代码添加 提交 ，假如需要写的提交是 “测试”，在测试之前添加表情，那么请用下面的代码

```csharp
git add .
git commit -m ":smile:测试"
```

可以看到，表情就是在`:`之间添加代表表情的提交，那么有哪些表情可以使用？

我不会让你一个个去试，我找到一个网站，提供了很多的表情

参见：https://www.webpagefx.com/tools/emoji-cheat-sheet/

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。