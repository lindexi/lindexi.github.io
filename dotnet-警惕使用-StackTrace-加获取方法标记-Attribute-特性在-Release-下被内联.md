
# dotnet 警惕使用 StackTrace 加获取方法标记 Attribute 特性在 Release 下被内联

大家都知道，在 dotnet 里的 Debug 下和 Release 下的一个最大的不同是在 Release 下开启了代码优化。启用代码优化，将会对生成的 IL 代码进行优化，同时优化后的 IL 也会有一些运行时的更多优化。内联是一个非常常用的优化手段，内联将会让 StackTrace 获取的调用堆栈存在 Debug 下和 Release 下的差异，从而导致获取方法标记的 Attribute 特性不能符合预期工作

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

这一个坑是来源于我所在团队开源的 CUnit (中文单元测试框架) 仓库的一次单元测试过程，我发现了在 Debug 下能通过测试，但是在 Release 下失败。详细请看： https://github.com/dotnet-campus/CUnit/actions/runs/3327386251/jobs/5502313113

核心原因是在 CUnit (中文单元测试框架) 仓库里使用了 StackTrace 的方式获取调用堆栈，通过调用堆栈获取各个方法，找到标记了 TestMethodAttribute 的方法，定位到标记是单元测试的方法

在 Release 下，发现找不到任何一个标记了 TestMethodAttribute 的方法。通过日志输出可以看到在 Debug 下和 Release 下的调用堆栈是不相同的。在 Release 下少了几个方法，刚好这几个方法里面就包含了一个标记了 TestMethodAttribute 的方法

其原因是在 Release 下默认开启了代码优化，在代码优化时，将会尝试内联一些函数，导致了调用堆栈中存在一些函数是看不到的，因为这些函数在实际运行过程中是不存在的，被内联到其他方法里面去了。换句话说，即使不是在 Release 下，只要开启了代码优化，那么都可能因为代码优化让某些函数被内联，从而让调用堆栈看起来不符合预期

因此，使用 StackTrace 获取调用堆栈，将在不同的环境下可能存在一些差异，导致逻辑不符合预期。如果再需要从方法上，获取方法标记的特性，那这个逻辑自然是不靠谱的

规避方法有两个：

第一个，那就是不要这么使用，找找其他的方法

第二个是，如果没有其他的方法，那可以考虑在明确需要获取某个特性的函数上，标记 `[MethodImpl(MethodImplOptions.NoInlining)]` 表示此函数不要被内联





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。