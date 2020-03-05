# cmd 如何跨驱动器移动文件夹

如果在命令行或 cmd 批处理文件通过 move 移动文件夹的时候，移动的文件夹是跨驱动器的，那么将会显示拒绝访问

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


解决通过 move 移动文件夹到不同的驱动器需要通过先复制文件夹到另一个驱动器，然后删除文件夹的方法解决

复制文件夹可以使用 xcopy 的方法

如复制文件，无论文件夹里面的文件是否隐藏文件。复制文件夹里面的子文件夹，无论子文件夹是否空的。忽略提示需要覆盖的文件，可以使用下面代码

```csharp
xcopy 原有的文件夹 移动到的文件夹  /H /E /Y 
```

然后通过 rd 删除文件夹，在 rd 可以使用 /s /q 除目录本身外，还将删除指定目录下的所有子目录和文件，安静模式，带 /S 删除目录树时不要求确认

```csharp
rd 原有的文件夹 /s /q
```

这样就可以做到移动文件夹到另一个服务器

在 cmd 对于参数是不区分大写和小写，如 `/s` 和 `/S` 是相同的

[xcopy-参数详解 - 疯狂的tiger - 博客园](https://www.cnblogs.com/yang-hao/p/6003308.html )

[Windows下使用CMD命令复制多层级目录 - 小灰笔记 - CSDN博客](https://blog.csdn.net/grey_csdn/article/details/77727591 )

我通过这个技术让希沃白板课件缓存文件夹可以放在有空间的盘，请看[希沃白板课件缓存文件夹迁移工具 解决C盘空间不足 - 分享汇 - 希沃论坛](http://bbs.seewoedu.cn/forum.php?mod=viewthread&tid=17342&extra= )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
