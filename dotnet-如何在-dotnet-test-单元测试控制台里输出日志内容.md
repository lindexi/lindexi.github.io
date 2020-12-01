
# dotnet 如何在 dotnet test 单元测试控制台里输出日志内容

我在协助小伙伴调试一个只有在 GitHub 的 Action 自动测试时才会炸的问题，而我发现默认的控制台输出是不会在 GitHub 的 Action 显示的，换句话说，在使用 dotnet test 时，代码里面使用的控制台输出不会进行输出

<!--more-->


<!-- CreateTime:2020/11/28 9:02:42 -->

<!-- 发布 -->

解决方法很简单，只需要在控制台输出的部分逻辑修改为 `Console.WriteLine` 而不是 `Debug.WriteLine` 方法

然后在 dotnet test 的命令加上 `-l "console;verbosity=detailed"` 代码，如下面代码


```
dotnet test --configuration release -l "console;verbosity=detailed"
```

这样就能在 GitHub 的 Action 进行单元测试时，输出对应的日志

为什么 `Debug.WriteLine` 方法没有输出？原因是 `--configuration release` 配置了不要让 Debug 下输出

更多请看 [Console.WriteLine calls during dotnet test are not emitted to the console on Windows · Issue #799 · microsoft/vstest](https://github.com/microsoft/vstest/issues/799 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。