# git 修改commit日期为之前的日期

我在之前修改了一个文件，但是没有commit，现在我想要commit，日期为那天的日期

git 修改日期的方法很简单，因为有一个命令`--date` 可以设置提交时间。

<!--more-->
<!-- csdn -->

使用git自定义时间的提交格式：

```csharp
git commit --date="月 日 时间 年 +0800" -am "提交"
```

如果我要把日期修改为 2016.5.7 那么我可以使用下面代码

```csharp
git commit --date="May 7 9:05:20 2016 +0800" -am "提交"
```

其中我希望大家知道的：


```csharp
January, Jan.
February, Feb.
March, Mar.
April, Apr.
May, May.
June, Jun.
July, Jul.
August, Aug.
September, Sep.
October, Oct.
November, Nov.
December, Dec.
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  