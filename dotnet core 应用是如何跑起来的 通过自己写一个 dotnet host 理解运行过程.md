# dotnet core 应用是如何跑起来的 通过自己写一个 dotnet host 理解运行过程

在上一篇博客是使用官方提供的 AppHost 跑起来整个 dotnet 程序。本文告诉大家在 dotnet 程序运行之前，是如果寻找 dotnet 运行时，是如何加载运行时然后跑起来业务端的 dll 文件的

<!--more-->
<!-- 不发布 -->

在上一篇博客告诉大家在 dotnet 的 AppHost 是如何做的，详细请看 [dotnet core 应用是如何跑起来的 通过AppHost理解运行过程](https://blog.lindexi.com/post/dotnet-core-%E5%BA%94%E7%94%A8%E6%98%AF%E5%A6%82%E4%BD%95%E8%B7%91%E8%B5%B7%E6%9D%A5%E7%9A%84-%E9%80%9A%E8%BF%87AppHost%E7%90%86%E8%A7%A3%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.html )

那如果我想要定制运行时的寻找路径呢？在 [dotnet core 应用是如何跑起来的 通过AppHost理解运行过程](https://blog.lindexi.com/post/dotnet-core-%E5%BA%94%E7%94%A8%E6%98%AF%E5%A6%82%E4%BD%95%E8%B7%91%E8%B5%B7%E6%9D%A5%E7%9A%84-%E9%80%9A%E8%BF%87AppHost%E7%90%86%E8%A7%A3%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.html ) 只是告诉大家如何定制咱的业务端的 dll 寻找路径

在运行 dotnet 程序的时候，在 windows 下需要通过 win32 的形式运行。而大家都知道，如果不开 AOT 等黑科技，咱构建输出的 dll 是 IL 中间格式的，但是可执行程序是只认机器码汇编的。如何从咱的 IL 逻辑到机器运行呢？今天咱来写这部分的逻辑，让整个 dotnet 跑起来的启动部分

需要知道，整个 dotnet 的启动机制是特别复杂的，本文只是告诉大家如何跑起来，也就是只是调用各个方法而已，细节部分我就不敢讲了

在开始之前，我推荐大家拉下我的代码到本地，通过自己更改实际修改代码可以理解整个 dotnet 的启动的 Native 部分逻辑

本文放在 [github](https://github.com/lindexi/lindexi_gd/tree/3e19bffc/HostWithMscoree ) 欢迎小伙伴访问

<!-- ![](image/dotnet core 应用是如何跑起来的 通过自己写一个 dotnet host 理解运行过程/dotnet core 应用是如何跑起来的 通过自己写一个 dotnet host 理解运行过程0.png) -->

![](http://image.acmx.xyz/lindexi%2F2020125131185274.jpg)

在代码仓库里面保护了两个模块，一个就是 SampleHost.vcxproj 包含的代码，这里就是 Native 的逻辑。另一个就是 ManagedLibrary 也就是咱 C# 的托管代码。下面让我来告诉大家这个仓库可以如何玩

先进入 ManagedLibrary 文件夹，双击执行 build.bat 文件，此时将会执行 dotnet 的发布命令，可以在发布之后进入 `HostWithMscoree\ManagedLibrary\bin\Debug\net5.0\win-x86\publish\` 文件夹，看到发布之后的内容

现在咱准备自己写一个 AppHost 应用，这个应用将支持从其他的路径找到运行时，然后执行 ManagedLibrary.dll 的逻辑。换句话说就是咱接下来的做法就是在 C 盘创建两个文件夹，分别是 `c:\lindexi\Code\HostWithMscoree\dll\` 和 `c:\lindexi\Code\HostWithMscoree\framework\` 文件夹

将 ManagedLibrary.dll 文件复制到 `c:\lindexi\Code\HostWithMscoree\dll\` 文件夹

将 `HostWithMscoree\ManagedLibrary\bin\Debug\net5.0\win-x86\publish\` 文件夹里面除了 ManagedLibrary 相关的文件外的其他文件复制到 `c:\lindexi\Code\HostWithMscoree\framework\` 文件夹

也就是说 ManagedLibrary.dll 的运行时框架都在 framework 文件夹，而 ManagedLibrary.dll 自己在 dll 文件夹里面。此时就需要在 dotnet 启动逻辑里面包含了去 `c:\lindexi\Code\HostWithMscoree\framework\` 文件夹寻找运行时的过程才能让 ManagedLibrary.dll 跑起来

请打开 `HostWithMscoree\SampleHost.sln` 文件，此时我期望你的 VS 上安装完成了 C++ 的负载，如果有提示缺少的部分，还请自行安装

打开 `HostWithMscoree\host.cpp` 文件，这里面将是整个核心的逻辑。这部分的逻辑相对清晰，但是知识点非常多，详细请看官方的 [Native hosting](https://github.com/dotnet/runtime/blob/master/docs/design/features/native-hosting.md ) 文档

在 `HostWithMscoree\host.cpp` 文件的 wmain 方法就是这个 SampleHost 的入口方法。可以看到这里被我定制了两个常量路径

```c++
int wmain(int argc, wchar_t* argv[])
{
    printf("Sample CoreCLR Host\n\n");
    // 替换下面的代码
    wchar_t* application = L"c:\\lindexi\\Code\\HostWithMscoree\\dll\\ManagedLibrary.dll";
    const wchar_t* coreCLRDirectory = L"c:\\lindexi\\Code\\HostWithMscoree\\framework\\";

    // 忽略代码
}
```

我在上面代码分开了 application 应用所在的文件夹路径，以及应用的运行时所在的文件夹路径，作为两个不同的常量路径（当然了，第一个实际上代码上不是常量，只是逻辑上是常量）用于在后续使用。请大家根据自己的需要更改路径

在 `c:\lindexi\Code\HostWithMscoree\framework\` 文件夹里面存放的就是 ManagedLibrary 项目里面除了 ManagedLibrary.dll 和 exe 等文件外的，其他文件，也就是说 framework 文件夹里面存放的是当前 ManagedLibrary.dll 的运行时和框架所有文件

而 application 对应的文件夹里面就只有 ManagedLibrary.dll 一个文件，因为其他的文件咱也不需要用到。如 ManagedLibrary.exe 其实就是 AppHost 文件，这个文件的功能就是作为 dotnet 的启动入口，而这个功能就是咱的 SampleHost 所要完成的功能。也就是说咱将使用 SampleHost 代替 AppHost 也就是 ManagedLibrary.exe 文件来作为 dotnet 的启动入口

配置完成之后，请按下 F5 运行起来。如果看到只是一闪而过，还请在适当地方添加断点哈，因为 C++ 程序在 VS 上没有做和 C# 一样的优化，在执行完成之后不退出控制台

如果能运行成功，可以看到如下界面

<!-- ![](image/dotnet core 应用是如何跑起来的 通过自己写一个 dotnet host 理解运行过程/dotnet core 应用是如何跑起来的 通过自己写一个 dotnet host 理解运行过程1.png) -->

![](http://image.acmx.xyz/lindexi%2F20201251316599609.jpg)

只有一句 这是在 dotnet 的输出 是跑在咱的 dotnet 应用里面的，其他都是 SampleHost 的

这样咱就完成了一个 dotnet 启动器，可以从指定的路径找到运行时和框架，然后运行指定路径的应用

在跑起来之后，可以先看看这个项目里面的代码和注释

咱按照代码的顺序，一步步告诉大家这是在做什么



本文的这个代码是微软官方的例子，但是微软在 dotnet 5 时删除了这个例子，因此需要在 [https://github.com/dotnet/samples](https://github.com/dotnet/samples) 项目里面切换 commit 到 a8804d38692d6c2a4bf9e78d0058edaf8c9cf955 才能找到本文的例子，本文的例子放在 `core/hosting/HostWithMscoree` 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
