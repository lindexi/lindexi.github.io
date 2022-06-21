# dotnet 调试应用启动闪退的方法

应用程序如果启动即闪退，那大部分时候日志模块还没初始化完成，很难通过应用自身的启动流程了解到应用启动失败的原因。本文来告诉几个不同的方法用来调查应用启动失败的原因

<!--more-->
<!-- CreateTime:2022/6/20 8:03:37 -->

<!-- 发布 -->

应用启动失败的原因可能有很多，例如系统环境问题，例如写个点逗比代码，例如调用某个带毒的库。如果应用启动失败，可以在开发环境上复现，那无疑是十分好的事情，因为咱可以使用开发环境强大的 VisualStudio 调试工具进行调试

## 使用 VisualStudio 调试应用启动失败

在有符号的配合下，使用 VisualStudio 定位应用软件启动失败在大多数时候都是比较轻松的。当然，没有符号的话，也没多少问题，至少可以快速定位到是哪个模块

使用 VisualStudio 定位应用软件启动失败的方法是让 VisualStudio 启动应用且进入调试模式。做法就是随便找一个 dotnet 6 的项目，当然，如果是所要调试的应用的对应版本的代码的项目那是最好的。点击设置调试属性，设置应用作为启动路径

在 VisualStudio 2022 下，打开设置调试属性的界面可以是在项目上进行右击，然后点击属性，找到调试页面，点击打开调试启动配置文件即可看到，如下图

![](http://image.acmx.xyz/lindexi%2F20226182047394478.jpg)

接着点击创建新配置文件，选择可执行文件

![](http://image.acmx.xyz/lindexi%2F20226182049315232.jpg)

接下来选择需要调试启动失败的应用的路径

![](http://image.acmx.xyz/lindexi%2F20226182050218037.jpg)

为了同时捕获一些本机异常，还请勾选“启用本机代码调试”也就是混合调试模式。本机异常包括 Window Runtime 抛出的异常，基础的 Win32 调用包含的非返回值的错误的异常，以及外部 C++ 等库的异常等

![](http://image.acmx.xyz/lindexi%2F2022618205318555.jpg)

为了提升调试的成功率，还请在 VisualStudio 设置里面，将所有的异常都打开进行捕获，同时关闭仅我的代码调试。打开所有异常捕获的方法是在 调试->窗口->异常设置 里面进行配置。简单来说就是将能打钩的全部打上，当然，你要是熟悉的话，那就少打钩一些咯，反正多打钩也没啥问题

关闭仅我的代码可以让你调试到一些被优化的代码。在咱 dotnet 的程序集里面，对 Debug 下和 Release 下最大的不同在于勾选了优化代码。如果勾选了仅我的代码调试，那将只调试 Debug 生成的程序集，而默认忽略对 Release 的程序集的记录。在大部分的调试下，这个模式都可以减少发布的程序集的干扰，可以更加方便调试业务代码。但是当前是在调试启动失败，启动失败可能是库的锅，需要调试发布的程序集，推荐关闭仅我的代码调试。关闭的方法是在 VisualStudio 的 工具-> 选项 -> 调试 里面，去掉 启用“仅我的代码” 的选项

![](http://image.acmx.xyz/lindexi%2F20226182113182818.jpg)

完成配置之后，在 VisualStudio 里面，选择刚才创建的新配置作为启动项进行启动

推荐是第一次调试可以快速过，看看是不是有异常触发，逐步去掉那些不影响启动异常的干扰，尝试找到导致启动失败的异常，即可进行快速定位

而启动失败还有一个隐藏的原因是写了逗比代码，自己退出的。那就需要自己进行调试，找到是哪个模块退出了应用，可以在第一次调试的时候，通过输出窗口找到应用的退出码是多少，辅助定位逻辑。如果退出码是一个零，那找找是不是存在 `Environment.Exit(0);` 类似的代码，可以全局进行字符串查找对应的代码。或者是 Main 函数执行完成，例如在 WPF 里面调用了 `Application.Current.Shutdown` 进行退出

在开发环境上遇到应用启动失败，大部分时候都可以在 VisualStudio 的帮助下快速定位到为什么启动失败

但是如果应用只是在用户的设备上才失败，那就没那么好玩了，接下来将告诉大家如何调试用户端的应用启动失败

## 使用 dnSpy 调试应用启动失败

在用户的设备上，如果应用启动失败了，如果此时应用自己的日志模块还没初始化完成，那也不用慌，系统的事件查看器可能可以帮忙到你。打开系统的事件查看器，里面也许记录了一些应用启动失败的原因，例如是系统环境问题，比如是系统缺少了某个库，或者是驱动问题。我之前很经常遇到的就是 WPF 应用启动失败是由显卡驱动导致的，不过显卡驱动问题基本上用不到多少的调试，稍微看一下就能看到了，系统的各个部分都会很奇怪

如何打开系统的事件查看器？在 Win10 下，右击开始菜单按钮，点击事件查看器即可打开。打开之后，大部分时候都可以先去看 Windows 日志里面的应用程序的日志，里面也许有记录应用的启动失败原因

但是有时候事件查看器记录的也很迷，如下面例子的启动失败的记录

系统记录了两条相关的错误日志，一条是 .NET Runtime 错误日志，如下内容

```
Application: KajijuniLiguqujokemka.exe
CoreCLR Version: 6.0.522.21309
.NET Version: 6.0.5
Description: The process was terminated due to an internal error in the .NET Runtime at IP 00007FF9DAEBDA03 (00007FF9DACF0000) with exit code c0000005.
```

另一条是 Application Error 日志，内容如下

```
错误应用程序名称: KajijuniLiguqujokemka.exe，版本: 1.0.0.0，时间戳: 0x62571213
错误模块名称: coreclr.dll，版本: 6.0.522.21309，时间戳: 0x625708f4
异常代码: 0xc0000005
错误偏移量: 0x00000000001cda03
错误进程 ID: 0x3814
错误应用程序启动时间: 0x01d882fdfe019fc7
错误应用程序路径: C:\lindexi\Code\lindexi\BeyajaydahifallChecheecaifelwarlerenel\KajijuniLiguqujokemka\bin\Debug\net6.0-windows\KajijuniLiguqujokemka.exe
错误模块路径: C:\Program Files\dotnet\shared\Microsoft.NETCore.App\6.0.5\coreclr.dll
报告 ID: 45232171-a61e-46fa-b80b-248ad12f5fef
错误程序包全名: 
错误程序包相对应用程序 ID: 
```

这两条日志没有能给咱很好的一个调试思路，只能说明应用确实挂了而已。不能说明是应用自己写了逗比代码，也不能证明是系统环境问题，也不能证明是调用库的问题。想要了解为什么，只能继续往下进行调试

通过 dnSpy 神器可以辅助在用户端进行调试。根本原因在于 VisualStudio 太庞大了，在用户端安装不太现实。但 dnSpy 是非常轻巧的，可以免安装使用。相当于在用户端跑一个轻巧的 VisualStudio 调试工具

支持 dotnet 6 版本的 dnSpy 下载地址请看 [支持 dotnet 6 的 dnSpy 神器版本](https://blog.lindexi.com/post/%E6%94%AF%E6%8C%81-dotnet-6-%E7%9A%84-dnSpy-%E7%A5%9E%E5%99%A8%E7%89%88%E6%9C%AC.html)

调试的思路和上文的使用 VisuslStudio 调试的差不多，有稍微一点不同的是，需要先将要调试的 Exe 拖入到 dnSpy 中，然后点击此 Exe 进行调试。同样需要勾选异常等

使用 dnSpy 调试还有一个好处是，可以无须任何符号即可进行调试，十分方便

## 使用 ProcDump 进行 DUMP 分析

但是如果应用的启动失败不是每次都复现的，是概率复现的，那就不好玩了。以上两个方法都是需要进行调试启动的，而大家都知道，调试模式下和非调试模式下是有差别的，例如多线程执行的差别。如果刚好启动是因为线程安全导致的问题，那么调试下也许是复现不到的。对于不是每次都失败的应用启动，进行调试是非常想砸键盘的，有时候调试的好好的，应用就启动成功了。有时候觉得没问题，按下继续，应用就启动失败了

或者是在用户端，用户有情绪了，不适合进行慢慢的调试。此时可以用到 ProcDump 工具辅助，在应用启动时候的时候，将失败时做一个 DUMP 文件，然后咱就可以将这个 DUMP 传回开发的设备上慢慢进行分析

这个 ProcDump 是微软极品工具箱的一个很有名的工具

官方下载地址： [https://docs.microsoft.com/zh-cn/sysinternals/downloads/procdump](https://docs.microsoft.com/zh-cn/sysinternals/downloads/procdump?WT.mc_id=WD-MVP-5003260)

根据官方文档可以了解到使用方法是在命令行使用如下参数，即可做到在应用因为异常挂掉自动捕获 DUMP 文件

```
procdump.exe -e -t -w -ma <进程名>
```

参数的含义如下

- `-e` : 当进程遇到未经处理的异常时写入转储
- `-t` : 进程终止时写入转储。如果应用启动失败是自己逗比或者某个库逗比调用了退出进程的方法，那也可以使用捕获到
- `-w` : 等待指定的进程启动。大部分时候都是先运行 ProcDump 工具，然后再启动应用，这样 ProcDump 相当于监控应用启动失败或退出。如此即可采用 ProcDump 启动进程调试应用启动闪退
- `-ma` : 获取的是 Full Dump 文件，也就是包含所有内容的 DUMP 文件，虽然这个 DUMP 比较大，但是调试会根据方便。如果传输过程比较难，而且开发者也熟悉调试 DUMP 可以换用 `-mm` 命名写入小型 DUMP 文件

假定需要调试的启动失败的应用是 `KajijuniLiguqujokemka.exe` 应用，那么执行的命令如下

```
procdump.exe -e -t -w -ma KajijuniLiguqujokemka
```

如此即可在应用启动闪退自动创建 DUMP 文件。创建好的 DUMP 文件可以采用 7z 工具压缩一下再传回开发机器，使用 7z 可以极大压缩 DUMP 文件，因为 DUMP 文件里面很多数据都是全 0 的

拿到 DUMP 文件之后，就需要开启 DUMP 调试了。最简单的 DUMP 调试是打开 VisualStudio 将 DUMP 文件拖进入，然后如开始的步骤先配置一下，然后点击使用混合进行调试即可

核心是看调用堆栈，和局部变量窗口，找到是哪个模块抛出异常或者退出。如果 VisualStudio 无法帮到你，那就只能换成 WinDbg 啦，不过这又是另外一个故事了

大家可以尝试使用我放在 [github](https://github.com/lindexi/lindexi_gd/tree/56318bf4ca4337539f65987cec2b057c1f9c6f8e/BeyajaydahifallChecheecaifelwarlerenel ) 的代码进行测试

更多请看 [dotnet 代码调试方法](https://blog.lindexi.com/post/dotnet-%E4%BB%A3%E7%A0%81%E8%B0%83%E8%AF%95%E6%96%B9%E6%B3%95.html )