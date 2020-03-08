# WPF 如何在应用程序调试启动

如果在一些无法使用源代码编译的电脑，调试一个exe无法启动，那么需要使用本文的技术。

<!--more-->
<!-- CreateTime:2019/6/11 9:32:35 -->


<!-- csdn -->
<!-- 标签：WPF，VisualStudio，调试 -->

首先打开 App.xaml.cs 然后在构造函数添加下面代码

```csharp
            System.Diagnostics.Debugger.Launch();

```

在启动时，就会打开调试器，于是就可以调试exe启动。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171030174351.jpg)

如果添加了上面的代码无法使用，那么可以判断现在是否调试，如果没有，就等待

```csharp
while (!System.Diagnostics.Debugger.IsAttached)
    {
    	System.Threading.Thread.Sleep(100);
    }
```

这时打开VS点击调试附加到进程，直到附加到进程程序才进行。

[https://stackoverflow.com/q/9896857/6116637](https://stackoverflow.com/q/9896857/6116637)

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017117185341.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  