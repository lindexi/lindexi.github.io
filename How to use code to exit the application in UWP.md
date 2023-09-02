# How to use code to exit the application in UWP

I will tell you how to exit the application in UWP by the code.

<!--more-->

We can call some static method to help us to exit the application.

The first method is as this code:

```csharp
            CoreApplication.Exit();

```

And the other one is 

```csharp
Application.Current.Exit();
```

If you are making the WPF application that you can use this code to exit the application.

```csharp
Application.Current.Shutdown();

```

And

```csharp
Environment.Exit(0);

```

If you find any problems, please contact me.

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
