# 如何在 CMD 启动的软件传入带空格的路径

在使用 CMD 命令的时候，会将传入的命令按照空格分为多个不同的命令，但是路径经常是带有空格的。特别是想将参数传入到通过命令行启动的软件里面，可以如何做？

<!--more-->
<!-- CreateTime:2019/5/25 9:31:46 -->

<!-- csdn -->

假如通过 CMD 命令启动我的一个放在 `C:\lindexi 是逗比` 的 Foo.exe 程序，那么可以通过下面代码启动

```
cmd.exe "C:\lindexi  是逗比\Foo.exe"
```

因为路径里面的 `C:\lindexi  是逗比\Foo.exe` 有空格，需要通过引号包含

如果我需要给 Foo.exe 传入参数，参数内容是 `foo` 那么需要通过 `/K` 或 `/C` 的命令将参数传入，在 `/K` 或 `/C` 后面的参数将会传入到执行的程序，而不是作为 CMD 的参数

通过 `/k` 可以在执行之后不退出 cmd 程序，通过 `/C` 可以在执行完程序之后就退出 cmd 程序

例如我需要将 `C:\林德熙 是逗比` 作为参数传入到 Foo.exe 那么下面代码执行的时候，因为传入 CMD 命令的路径带来空格，需要通过引号包含

但是在 CMD 里面传入多个带引号的路径会被作为多个传入 CMD 的启动参数，刚好参数路径不是可以执行的文件

```
C:\user\lindexi> cmd /k "C:\lindexi  是逗比\Foo.exe" "C:\林德熙 是逗比"
文件名、目录名或卷标语法不正确
```

在传入的参数里面存在空格，需要使用最外层的一个引号包含

```csharp
cmd /k " xx.exe xx参数 "
```

通过这个方法可以解决 cmd 不认路径带空格，和不认使用引号包含的路径

```
cmd /k " "C:\lindexi  是逗比\Foo.exe" "C:\林德熙 是逗比" "
```

[cmd.exe 的命令行启动参数（可用于执行命令、传参或进行环境配置） - walterlv](https://walterlv.com/post/cmd-startup-arguments.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
