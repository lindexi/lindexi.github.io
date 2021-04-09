
# Windows 下的 WPF 开发 调试应用程序在什么时机加载了 Dll 模块

在尝试优化性能的时候，如何可以了解到在应用程序启动的过程中，在什么步骤开始加载了某些 Dll 文件

<!--more-->


<!-- 发布 -->

在 VisualStudio 的 调试->窗口->模块 可以看到当前应用程序加载的所有模块，也就是应用程序加载了哪些 Dll 文件

一个调试方法是在合适的逻辑里面添加断点，或者在软件启动完成之后，通过模块了解应用程序加载了哪些 DLL 文件，从而了解应用程序启动慢是否因为加载了不应该加载的模块

在 dotnet 里面，可以通过辅助的代码了解是在哪些模块加载了 DLL 文件，例如我在调试的 [SVG 库](https://github.com/dotnet-campus/SharpVectors) 是在哪个模块加载的，我不期望在启动的过程中有加载 SVG 相关的 DLL 文件，那么我可以如何了解到是在应用程序的哪个逻辑里面加载的？可以通过在应用程序的主函数里面添加如下代码用来在加载到 [SharpVectors](https://github.com/dotnet-campus/SharpVectors) 模块进入断点

```csharp
        [STAThread]
        static void Main(string[] args)
        {
            AppDomain.CurrentDomain.AssemblyLoad += CurrentDomain_AssemblyLoad;
        }

        private static void CurrentDomain_AssemblyLoad(object sender, AssemblyLoadEventArgs args)
        {
            if (args.LoadedAssembly.FullName.Contains("SharpVectors"))
            {
                Debugger.Break();
            }
        }
```

如果是在 WPF 默认的应用里面，没有 Main 函数，那么写到 App 的构造函数也可以

```csharp
public App()
{
    AppDomain.CurrentDomain.AssemblyLoad += CurrentDomain_AssemblyLoad;
}
```

在进入 CurrentDomain_AssemblyLoad 函数加载到 SharpVectors 的模块的时候，将会进入断点。通过调用堆栈，可以了解到是在访问到哪个业务逻辑需要加载的，然后再调试这个业务逻辑是否需要放在启动的过程





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。