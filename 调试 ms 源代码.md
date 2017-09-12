# 调试 ms 源代码

如果需要调试 WPF 源代码或框架源代码，那么需要使用 DotPeek。

<!--more-->
<!-- csdn -->

首先需要下载 dotPeek ，可以到官网下载 [dotPeek: Free .NET Decompiler & Assembly Browser by JetBrains](https://www.jetbrains.com/decompiler/) 还可以到 csdn 下载

首先打开 dotPeek 然后点击启动符号服务器，所有符号。

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201798184736.jpg)

然后点击工具设置，可以看到这个页面

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201798185154.jpg)

然后打开 VS 工具选项，在调试设置符号，刚才已经复制了，现在添加就好

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20179819559.jpg)

然后还需要去掉微软的服务和本地缓存

然后写一个呆磨进行测试

现在就可以开始调试框架源代码了

只需要在一些函数使用断点，然后堆栈跳转，假如我在 MouseDown 写一个断点，在触发按下，点击堆栈，可以看到外部代码。右击外部代码显示，这样就可以看到 垃圾wr 做的，双击他，可以跳到一个页面，点击加载就可以。

这时候可以看到 dotPeek 在反编译，这个时间比较长，需要去做一些你喜欢做的事情，回来就可以发现 dotPeek 反编译好而且你看到 ms 源代码，这时候可以尝试源代码断点，但是不是所有地方都可以断点。

如果你发现无法进入代码，那么尝试安装 Resharper ，如果还是不行，那么需要问一下，是不是使用 UWP ，因为现在我尝试 UWP 还没有成功。

如果还是无法成功，不要来问我，我教了几个小伙伴，有几个是没法进入代码，使用方法都一样，我自己去他电脑弄了，结果我无法进入。

