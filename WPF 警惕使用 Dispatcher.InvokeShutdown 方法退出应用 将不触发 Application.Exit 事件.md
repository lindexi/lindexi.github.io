# WPF 警惕使用 Dispatcher.InvokeShutdown 方法退出应用 将不触发 Application.Exit 事件

这是一个比较让人困惑的一个点，我一直都以为 Dispatcher.InvokeShutdown 和 Application.Current.Shutdown 是完全等价的。但是后面发现了其实这两者还是有些不同的，感觉上是 Dispatcher.InvokeShutdown 系列方法有点点设计的问题，太过于为了让框架内的代码解耦导致了让上层开发者困惑。 推荐在退出应用时，尽量调用的是 Application.Current.Shutdown 方法，而不是 Dispatcher.InvokeShutdown 系列方法

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

先来说说 Dispatcher.InvokeShutdown 和 Application.Current.Shutdown 的区别。在调用 Application.Current.Shutdown 方法时，将会触发 `Application.Current.Exit` 事件，以及在框架内调用 `Dispatcher.InvokeShutdown` 方法，且触发 `Dispatcher.ShutdownFinished` 事件

在调用 `Dispatcher.InvokeShutdown` 系列方法，包括 `Dispatcher.BeginInvokeShutdown` 等时，将不触发 `Application.Exit` 事件，只触发 `Dispatcher.ShutdownFinished` 事件

如此可以看到，调用 Application.Current.Shutdown 方法是包含了 `Dispatcher.InvokeShutdown` 方法的，能比较符合开发者预期的触发各个退出事件

我写了一个 Demo 用来方便让大家了解这两个方法触发的差别，代码放在[github](https://github.com/lindexi/lindexi_gd/tree/fe31b73707f46f753976c276bc3aea74e85f9127/HineakemnerFeceqerhai) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/fe31b73707f46f753976c276bc3aea74e85f9127/HineakemnerFeceqerhai) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin fe31b73707f46f753976c276bc3aea74e85f9127
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin fe31b73707f46f753976c276bc3aea74e85f9127
```

获取代码之后，进入 HoyebenawlerWegemnardicheba 文件夹