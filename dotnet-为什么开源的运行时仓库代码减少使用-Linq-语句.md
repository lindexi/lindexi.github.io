
# dotnet 为什么开源的运行时仓库代码减少使用 Linq 语句

在 dotnet 开源的 runtime 运行时仓库里面，有微软的大佬说运行时仓库的代码应该减少使用 Linq 语句，那这又是为什么呢

<!--more-->


<!-- CreateTime:2020/8/24 14:21:49 -->



微软的 [Jan Kotas](https://github.com/jkotas) 大佬说了下面这段话，大概意思就是减少在运行时库里减少对 Linq 的使用

```
Linq maybe saves some allocations, but it comes with other overheads and much larger static footprint so it is not a clear winner. We tend to avoid Linq in the runtime libraries implementation.
```

而 [Günther Foidl](https://github.com/gfoidl ) 小伙伴就帮我问了一句为什么，难道是将会让单文件的体积，也就是输出的二进制文件体积比较大？

其实本质原因是启动时间，因此 Linq 将会需要创建很多泛形的类型

```
Startup time too. Linq tends to create a lot of generic type instantiations.
```

详细还请看 GitHub 的对话 [https://github.com/dotnet/runtime/pull/41137#discussion_r474742180](https://github.com/dotnet/runtime/pull/41137#discussion_r474742180)

因此在业务层依然可以使用 Linq 的，放心，没有性能问题

只是运行时库想要减少 JIT 创建泛形的类型的时间，因此减少使用而已

当然，本文只是裁几段话，没有很具体上下文含义。因此还请小伙伴阅读原文 [Reduce memory allocations for Process.GetProcessesByName by Serg046 · Pull Request #41137 · dotnet/runtime](https://github.com/dotnet/runtime/pull/41137 )

上面这个 PR 其实是我提出的一个问题，在调用 GetProcessesByName 的时候，是否可以减少一些内存的分配。尽管在获取进程的时候，性能是在获取的本机代码，但是多申请的内存是影响未来。这个意思是在调用这个方法的代码了解到这里的性能比较渣，因此将会有预期。而申请的内存，需要后续进行内存释放，这不是预期的，因此多申请内存影响的是之后。详细请看 [Can the GetProcessesByName method reduce the number of arrays and Process objects created? · Issue #40768 · dotnet/runtime](https://github.com/dotnet/runtime/issues/40768 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。