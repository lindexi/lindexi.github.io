# dotnet 获取程序所在路径的方法

在 dotnet 有很多方法可以获取当前程序所在的路径，但是这些方法获取到的路径有一点不相同，特别是在工作路径不是当前的程序所在的路径的时候

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


通过下面几个方法都可以拿到程序所在的文件夹或程序文件

- AppDomain.CurrentDomain.BaseDirectory 当前程序域寻找 dll 的文件夹
- Environment.CurrentDirectory 当前工作文件夹
- Assembly.GetCallingAssembly().Location 调用当前函数的函数的程序集的文件
- Assembly.GetEntryAssembly().Location 入口函数程序集所在的文件
- Assembly.GetExecutingAssembly().Location 包含当前代码的程序集的文件
- Directory.GetCurrentDirectory 当前工作文件夹

如写一个 SetereBojerhis 控制台程序，将这个程序放在 `D:\lindexi\dotnet 获取程序所在路径的方法\` 然后双击运行这个程序，可以看到下面代码

```csharp
AppDomain.CurrentDomain.BaseDirectory=D:\lindexi\dotnet 获取程序所在路径的方法\
Environment.CurrentDirectory=D:\lindexi\dotnet 获取程序所在路径的方法
Assembly.GetCallingAssembly().Location=D:\lindexi\dotnet 获取程序所在路径的方法\SetereBojerhis.exe
Assembly.GetEntryAssembly().Location=D:\lindexi\dotnet 获取程序所在路径的方法\SetereBojerhis.exe
Assembly.GetExecutingAssembly().Location=D:\lindexi\dotnet 获取程序所在路径的方法\SetereBojerhis.exe
Directory.GetCurrentDirectory()=D:\lindexi\dotnet 获取程序所在路径的方法
AppDomain.CurrentDomain.SetupInformation.ApplicationBase=D:\lindexi\dotnet 获取程序所在路径的方法\
Process.GetCurrentProcess().MainModule.FileName=D:\lindexi\dotnet 获取程序所在路径的方法\SetereBojerhis.exe
```

虽然看起来大多数的值都是相同的，但是还是有很多区别

## Assembly.GetCallingAssembly

获取调用这个函数的函数，如 Foo 函数里面调用了 `Assembly.GetCallingAssembly` 方法，那么将会返回调用 Foo 函数的函数所在程序集的文件路径

如存在程序集 A1 里面的 M1 方法，在 M1 方法调用 `Assembly.GetCallingAssembly` 方法。此时在程序集 A2 的 M2 方法调用了 M1 那么将会返回 M2 方法所在的程序集 A2 的文件

但是还有一个要求是 M1 方法不是内联到 M2 方法里面，如果进行内联，那么会让实际的 IL 在 M2 里面，也就是相当于是 M2 方法里面调用 `Assembly.GetCallingAssembly` 方法，不让一个方法作为内联可以使用 MethodImplOptions 特性

```csharp
  [MethodImpl (MethodImplOptions.NoInlining)]
  public static void OtherMethod () 
  {
      //这个方法将不会被内联
  }
```

## Assembly.GetEntryAssembly

获取入口程序集，一般的入口程序集就是包含 Main 函数的程序集，一个程序里面是可以存在多个 Main 函数，具体调用哪个可以在编译的时候指定，详细请看 [.NET/C# 中你可以在代码中写多个 Main 函数，然后按需要随时切换 - walterlv](https://walterlv.com/post/write-multiple-main-and-related-startup-codes.html )

通过在任意代码调用 Assembly.GetEntryAssembly 可以拿到当前调用的入口函数所在的程序集

但是这个方法相对使用的性能比较多，如果在调用 `Assembly.GetEntryAssembly` 方法所在的程序集和入口函数在不同程序集，那么性能将会比较差

同时如果是由非托管调用的函数，也就是入口函数不是托管代码那么调用 `Assembly.GetEntryAssembly` 将会返回空


## Assembly.GetExecutingAssembly

获取当前运行代码的程序集，如我在 Foo 方法调用 `Assembly.GetExecutingAssembly` 那么将会返回调用的代码所在的程序集

```csharp
   static void Foo()
   {
      // Get the currently executing assembly.
      Assembly currentAssembly = Assembly.GetExecutingAssembly();
      Console.WriteLine("Currently executing assembly:");
      Console.WriteLine("   {0}\n", currentAssembly.FullName);

   }
```

## 性能

已经几个获取方法的性能对比



|                               Method |       Mean |     Error |    StdDev |     Median |
|------------------------------------- |-----------:|----------:|----------:|-----------:|
|  AppDomainCurrentDomainBaseDirectory |   781.5 ns | 19.489 ns | 23.200 ns |   781.4 ns |
|          EnvironmentCurrentDirectory |   497.8 ns | 10.076 ns | 25.464 ns |   486.2 ns |
|   AssemblyGetCallingAssemblyLocation | 3,550.6 ns | 20.228 ns | 17.932 ns | 3,555.4 ns |
|     AssemblyGetEntryAssemblyLocation | 2,783.2 ns | 33.407 ns | 31.249 ns | 2,791.1 ns |
| AssemblyGetExecutingAssemblyLocation | 3,021.7 ns | 32.517 ns | 30.416 ns | 3,018.8 ns |
|         DirectoryGetCurrentDirectory |   472.2 ns |  3.871 ns |  3.621 ns |   471.4 ns |

## AppDomain.CurrentDomain.SetupInformation

关于 AppDomain.CurrentDomain.SetupInformation 和 Process.GetCurrentProcess().MainModule.FileName 的方法请看

[三种方法获取可执行程序的文件路径（.NET Core / .NET Framework） - walterlv](https://walterlv.com/post/get-current-executable-file-path.html#%E4%BD%BF%E7%94%A8%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E5%9F%9F%E4%BF%A1%E6%81%AF%E8%8E%B7%E5%8F%96 )

参考文档

[Assembly.GetEntryAssembly Method (System.Reflection)](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.assembly.getentryassembly?view=netframework-4.7.2 )

[Assembly.GetExecutingAssembly Method (System.Reflection)](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.assembly.getexecutingassembly?view=netframework-4.7.2 )

[Assembly.GetCallingAssembly Method (System.Reflection)](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.assembly.getcallingassembly?view=netframework-4.7.2#System_Reflection_Assembly_GetCallingAssembly )

[三种方法获取可执行程序的文件路径（.NET Core / .NET Framework） - walterlv](https://walterlv.com/post/get-current-executable-file-path.html#%E4%BD%BF%E7%94%A8%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E5%9F%9F%E4%BF%A1%E6%81%AF%E8%8E%B7%E5%8F%96 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
