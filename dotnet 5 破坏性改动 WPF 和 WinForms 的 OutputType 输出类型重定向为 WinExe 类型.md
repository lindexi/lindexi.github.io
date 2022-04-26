# dotnet 5 破坏性改动 WPF 和 WinForms 的 OutputType 输出类型重定向为 WinExe 类型

官方团队为了防逗比，默认将 OutputType 输出类型重定向为 WinExe 类型，这样能解决很多新手遇到的 WPF 项目显示出黑框控制台界面问题。而对于一些老司机来说，这个特点反而有点迷，特别是在尝试打开控制台调试的时候

<!--more-->
<!-- CreateTime:2020/12/14 8:36:58 -->

<!-- 发布 -->

在开始之前，先复习一下 OutputType 这个属性的作用，这个属性告诉了 SDK 程序集输出的文件是什么。可以选的值是 Library 类库和 Exe 控制台程序以及 WinExe 带界面的程序

而 Exe 和 WinExe 的差别对于 WPF 和 WinForms 程序来说就是是否在启动的时候显示出控制台出来。而在 dotnet 5 的时候，只要引用了 WPF 或 WinForms 的 SDK 内容，那么 SDK 默认将会重定向 OutputType 为 WinExe 类型

这个行为也能说明为什么引用了 WPF 之后，运行程序之后啥都没发生。因为如果原先是控制台程序，引用了 WPF 之后不显示控制台了，但是项目原先又没有显示如任何的窗口，就好像应用啥都不做一样

如果想要禁用这个默认的行为，只需要在 csproj 上添加 DisableWinExeOutputInference 属性修改为 true 就可以了，代码如下

```xml
    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <DisableWinExeOutputInference>true</DisableWinExeOutputInference>
        <TargetFramework>net5.0-windows</TargetFramework>
        <UseWPF>true</UseWPF>
    </PropertyGroup>
```

更多细节请看官方文档 [Breaking change: OutputType set to WinExe for WPF and WinForms apps](https://docs.microsoft.com/en-us/dotnet/core/compatibility/windows-forms/5.0/automatically-infer-winexe-output-type?WT.mc_id=WD-MVP-5003260)

至于 OutputType 是 Exe 还是 WinExe 类型的更多细节，请参阅 [dotnet core 通过修改文件头的方式隐藏控制台窗口](https://blog.lindexi.com/post/dotnet-core-%E9%80%9A%E8%BF%87%E4%BF%AE%E6%94%B9%E6%96%87%E4%BB%B6%E5%A4%B4%E7%9A%84%E6%96%B9%E5%BC%8F%E9%9A%90%E8%97%8F%E6%8E%A7%E5%88%B6%E5%8F%B0%E7%AA%97%E5%8F%A3.html)
 
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
