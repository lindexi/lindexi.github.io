# WPF 开发自动删除软件

我在写一个测试的工具，需要在用户的电脑使用，但是需要在运行之后 3 天内删除。这个功能是因为我是金鱼但是自己忘记删除了，但是可能需要多次使用，所以就需要让软件自动删除。

<!--more-->
<!-- CreateTime:2018/9/2 14:51:48 -->

<!-- csdn -->

我的方法是使用调用 bat 删除自己的方法，首先写一个可以删除自己的 bat 文件

```csharp
@echo off

timeout /t 5 > nul

@RD /S /Q  %cd%

@exit
```

将代码复制到 bat 文件，运行就可以删除 bat 所在的文件夹

所以只需要在代码里自动调用这个程序就可以。

![](https://i.loli.net/2018/09/02/5b8b8881f1a33.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
