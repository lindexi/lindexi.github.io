# dotnet 使用 ClearScript 执行 VBScript 和 JS 代码 无需浏览器

小伙伴都知道，使用 JS 的坑在于执行效率过低，速度过慢。如果是在客户端中，还开启一个浏览器，整个应用程序就会特别重。有没有什么方法可以让 dotnet 作为容器，执行 JScript 代码而不需要浏览器？其实有的，因为 VBScript 和 JS 代码都很好解析，所以有 ClearScript 支持使用 dotnet 解析代码执行

<!--more-->
<!-- CreateTime:2020/2/5 11:28:28 -->

<!-- 发布 -->

如何将 VBScript 和 JS 代码编译为 IL 或如何在 C# 和 dotnet 中嵌入 js 代码是不靠谱的，因为 js 语言设计如此，是用来在 基组模块 层运行的语言，也就是这是一个解析型的脚本，更好地做法是给他实现一个解析库。通过解析库将代码解析为数据，在根据数据调用已经写好的代码，这样就能实现在 dotnet 中运行 js 代码了。实现这个方式有两个不同方法，第一个方法使用的最多的，就是在 dotnet 中添加一个浏览器让他执行代码，这个方法的缺点就是性能特别渣，无论是内存占用或 CPU 占用都是特别渣。我作为性能优化组的逗比，只能去寻找第二条路径，毕竟需要照顾一些只会写 js 的大佬。第二个方法是通过 dotnet 解析器解析 js 代码的方法，推荐使用 ClearScript 库，这个库十分好用，可以给 js 注入执行的类或实例包括库，这样可以让贫瘠的 js 可以用到更多的 dotnet 的强大的库。同时可以杂私货，注入自己实现的类等。因为可以选择注入执行的类，所以可以让一些如文件访问相关的库不要直接注入，而是注入自己封装的文件访问相关的库，解决不安全的 js 带毒的问题

先创建一个 dotnet 项目，然后通过 NuGet 安装 [Microsoft.ClearScript](https://www.nuget.org/packages/Microsoft.ClearScript )

然后就可以在代码中运行 VBScript 代码或 js 代码了


```csharp
                // expose a host type
                engine.AddHostType("Console", typeof(Console));

                engine.Execute("Console.WriteLine('{0} is an interesting number.', Math.PI)");
```

可以注入自己创建实例

```csharp
                // expose a host object

                engine.AddHostObject("random", new Random());

                engine.Execute("Console.WriteLine(random.NextDouble())");
```


可以给其他库

```csharp
               engine.AddHostObject("lib", new HostTypeCollection("mscorlib", "System.Core"));

                engine.Execute("Console.WriteLine(lib.System.DateTime.Now)");

                // create a host object from script

                engine.Execute(@"

        birthday = new lib.System.DateTime(2007, 5, 22);

        Console.WriteLine(birthday.ToLongDateString());

    ");
```



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
