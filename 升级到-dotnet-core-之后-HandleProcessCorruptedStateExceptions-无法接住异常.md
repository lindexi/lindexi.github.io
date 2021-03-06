
# 升级到 dotnet core 之后 HandleProcessCorruptedStateExceptions 无法接住异常

这是 dotnet core 的破坏性改动之一，在 dotnet framework 里面，可以使用 HandleProcessCorruptedStateExceptionsAttribute 接住非托管层抛出的异常，如 C++ 异常等。但是这个功能在 dotnet core 下存在行为的变更，从 .NET Core 1.0 开始，损坏进程状态异常无法由托管代码进行处理。 公共语言运行时不会将损坏进程状态异常传递给托管代码

<!--more-->


<!-- CreateTime:2021/1/25 8:48:37 -->

<!-- 发布 -->

如果逻辑代码完全使用 C# 实现，那么应用程序可以称为是安全的。这里的安全指的是内存安全。这是 dotnet 的一个优势，在于异常处理上，和 C++ 等的异常处理不同的是，很少会有异常能让整个程序闪退。可以很方便在应用程序里面接住软件运行异常，然后通过各个方法让软件继续执行

但如果 C# 调用了 C++ 的库，那就不好玩了，这就意味着如果 C++ 的库如果实现不够好的话，那么这个库是能带着整个应用程序闪退的。而有趣的是，其实我到现在还没遇到几个团队写出的 C++ 库是稳定的，基本上通过我的 DUMP 分析可以看到，每多加一个 C++ 库，软件的稳定性就下降一半。好在，有一些 C++ 库抛出来的异常，咱勉强还是能接住的，至少不会让整个应用程序就闪退了

接住 C++ 异常的其中一个方法就是通过 HandleProcessCorruptedStateExceptions 特性，在方法上面标记 HandleProcessCorruptedStateExceptions 特性，此时在方法里面使用 try catch 是可以接住大部分的 C++ 异常的，如 System.AccessViolationException 异常

请看下面代码

```csharp
        [HandleProcessCorruptedStateExceptions]
        static void Main(string[] args)
        {
            try
            {
                Console.WriteLine(HeederajiYeafalludall());
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
            }
        }

        [DllImport("BeyajaydahifallChecheecaifelwarlerenel.dll")]
        static extern Int16 HeederajiYeafalludall();
```

上面代码的 HeederajiYeafalludall 方法是由 BeyajaydahifallChecheecaifelwarlerenel.dll 提供的，这是一个由 C++ 写的库，在这里面的实现将会出现越界

```C++
extern "C" __declspec(dllexport) int HeederajiYeafalludall() 
{
    int* p = (int*)123;
    while (true)
    {
        *p = 123;
        p++;
    }

    return 123;
}
```

在标记了 HandleProcessCorruptedStateExceptionsAttribute 特性之后，将可以看到断点能进入到 catch 代码里，而且程序不会闪退

但是这个机制在 dotnet core 就跑不起来了，根据 [从 .NET Framework 到 .NET Core 的中断性变更](https://docs.microsoft.com/zh-cn/dotnet/core/compatibility/fx-core?WT.mc_id=WD-MVP-5003260) 文档，可以看到在 .NET Core 1.0 开始，损坏进程状态异常无法由托管代码进行处理，将上面的 C# 代码切换到 dotnet core 下执行，此时将会发现不会进入到 catch 的代码，应用程序将会退出

大家可以尝试使用我放在 [github](https://github.com/lindexi/lindexi_gd/tree/9bf58ca4/BeyajaydahifallChecheecaifelwarlerenel ) 的代码进行测试，切换框架为 .NET Framework 和 .NET Core 比较这里的行为







<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。