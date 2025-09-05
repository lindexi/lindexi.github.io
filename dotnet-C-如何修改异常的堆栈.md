
# dotnet C# 如何修改异常的堆栈

异常的堆栈信息，通常都是在抛出的时候自动记录当前的异常堆栈信息。在某些特殊情况下，期望修改异常堆栈，此时需要用到 dotnet 内置的 ExceptionDispatchInfo 机制

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

实现方法非常简单，通过 `ExceptionDispatchInfo.SetRemoteStackTrace` 方法即可修改异常堆栈，如以下示例

```csharp
var exception = new ArgumentException();
ExceptionDispatchInfo.SetRemoteStackTrace(exception, $@"栽赃的堆栈.{nameof(Foo)}.{nameof(Foo.F1)}
Main");
Console.WriteLine(exception);
```

此时控制台输出内容大概如下

```
System.ArgumentException: Value does not fall within the expected range.
栽赃的堆栈.Foo.F1
Main
--- End of stack trace from previous location ---
```

如果想要对外抛出此加工后的异常，直接使用 `throw exception;` 扔掉即可，异常打印信息大概如下

```
System.ArgumentException: Value does not fall within the expected range.
栽赃的堆栈.Foo.F1
Main
--- End of stack trace from previous location ---
   at Program.<Main>$(String[] args)
```

或者使用 `ExceptionDispatchInfo.Throw` 扔也可以，代码示例如下

```csharp
    ExceptionDispatchInfo.Throw(exception);
```

异常打印信息大概如下

```
System.ArgumentException: Value does not fall within the expected range.
栽赃的堆栈.Foo.F1
Main
--- End of stack trace from previous location ---
   at Program.<Main>$(String[] args)
```

可见此时 `throw exception;` 和 `ExceptionDispatchInfo.Throw(exception);` 行为接近。正常来说会使用到 `ExceptionDispatchInfo.Throw` 方法的，都是针对于已经被抛出过的异常，进行重新捕获，延迟抛出的情况，详细请看 [使用 ExceptionDispatchInfo 捕捉并重新抛出异常 - walterlv](https://blog.walterlv.com/post/exceptiondispatchinfo-capture-throw.html )

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/ecffc754e5ae60028556594d962d39a42d8b6bc0/Workbench/FehallladaichaWhearjechelra) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/ecffc754e5ae60028556594d962d39a42d8b6bc0/Workbench/FehallladaichaWhearjechelra) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin ecffc754e5ae60028556594d962d39a42d8b6bc0
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin ecffc754e5ae60028556594d962d39a42d8b6bc0
```

获取代码之后，进入 Workbench/FehallladaichaWhearjechelra 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。