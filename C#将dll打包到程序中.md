# C＃ 将dll打包到程序中

本文告诉大家如何把 dll 打包到程序中。很多时候的 软件 在运行的时候需要包括很多 dll 或其他的文件，这样的软件在给其他小伙伴，就需要做一个压缩包，或者用安装软件。这样感觉不太好，所以本文告诉大家一个方法，把所有的 dll 放在一个文件，于是把自己的软件给小伙伴就只需要给他一个程序。

<!--more-->
<!-- CreateTime:2018/4/29 9:43:22 -->

<div id="toc"></div>

## ILMerge

首先下载[ ILMerge ](https://www.microsoft.com/en-us/download/details.aspx?id=17630 )

然后安装，感觉安装很简单

假如有 1.exe 和 1.dll 准备把 1.dll 合并到 2.exe 那么可以使用下面代码

```csharp
ilmerge  /target:exe /out:E:\2.exe /log E:\1.exe /log E:\1.dll /targetplatform:v4
```

这里的 target 为目标平台

out 就是输出的文件

log 就是准备合并的dll

执行代码就可以拿到 2.exe 直接把这个文件给小伙伴，他就不需要使用压缩包，直接打开 2.exe 就不会说找不到库。

参见：http://www.cnblogs.com/blqw/p/LoadResourceDll.html

[ILMerge将源DLL合并到目标EXE - HackerVirus - 博客园](http://www.cnblogs.com/Leo_wl/p/7792151.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
