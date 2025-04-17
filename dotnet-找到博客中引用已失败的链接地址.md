
# dotnet 找到博客中引用已失败的链接地址

在我的博客里面会添加很多引用，但是有一大部分的链接失修，访问的时候访问不到了，或者需要更新。于是我写了一个工具，可以协助找到所有的已失败的链接

<!--more-->


<!-- CreateTime:7/2/2020 3:36:30 PM -->



本文用到工具所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/5e17bc953f9e71de9d980ab844e7eceaa72234c8/ReanuyawnicayhiFawcerecheca) 欢迎小伙伴访问

使用方法是在参数传入博客所在的文件夹，此时将会找到所有最顶层的博客文件，接着工具将会使用正则 `@"([a-zA-z]+://[^\s^:^)^""]*)"` 找到所有的链接，然后尝试访问一下

如果链接不能返回 200 那么输出这个博客文件名和链接

```
ReanuyawnicayhiFawcerecheca C:\博客
```

可以在 ReanuyawnicayhiFawcerecheca.exe 所在文件夹找到 Log.txt 文件，里面将会是控制台输出的内容，内容如下

```
2020-07-02 09:15:09.850 [start] C:\博客\C# 反射调用私有事件.md
2020-07-02 09:15:12.053 C# 反射调用私有事件.md https://walterlv.com/post/add-event-handler-using-reflection.html 404
2020-07-02 09:15:12.311 C# 反射调用私有事件.md https://walterlv.com/post/create-delegate-to-improve-reflection-performance.html 404
2020-07-02 09:15:12.519 C# 反射调用私有事件.md https://walterlv.com/uwp/2017/09/21/reflection-using-dotnet-native-runtime-directive.html 404
2020-07-02 09:15:12.729 C# 反射调用私有事件.md https://walterlv.com/post/handle-ref-or-out-arguments-using-reflection.html 404
2020-07-02 09:15:12.936 C# 反射调用私有事件.md https://walterlv.com/post/design-a-cache-pool.html 404
2020-07-02 09:15:13.122 [end] C:\博客\C# 反射调用私有事件.md
```

这样小伙伴就能知道有哪些博客需要修改





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。