# WPF 编译为 AnyCPU 和 x86 有什么区别

本文告诉大家，编译为 AnyCpu 和 AnyCPU（Prefer 32-bit）和 x86 有什么区别

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->

## x86

编译为 32 位的程序，如果程序运行的机器是 32 位还是 64 位，程序运行都是 32 位，但是如果在 ARM 下，x86 程序无法运行

## AnyCPU

如果在 x86 系统下，运行就是 32 位程序，如果是 64 位系统下，运行就是 64 位程序。如果在 ARM 下运行，就是指定的 ARM 可以运行的程序。

## AnyCPU(Prefer 32-bit)

这是在右击属性，选择首选32位才会使用的方法，必须使用 .net framework 4.5 以上才可以使用。在这个编译下，程序运行都是 32 位。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201712151723520171225151314.jpg)

在 32 位系统下，运行 32 位程序

在 64 位系统下，运行 32 位程序，但是可以获得 4G 内存

在 ARM 下，运行 32 位程序

如果使用 AnyCPU 那么编译为 IL 是不需要加平台，程序在机器运行才判断机器平台，自动编译为机器可以运行的程序。

那么 AnyCPU(Prefer 32-bit) 和 x86 有什么区别？实际上在 ARM 系统，只能使用 AnyCPU(Prefer 32-bit) 运行 32 位程序，如果选择 x86 就无法运行。

为什么需要在 64 位的设备使用 AnyCPU(Prefer 32-bit)，因为如果存在一些库只能在 32位程序运行，那么就需要运行的程序是 32 位，所以需要使用这个方法。

如果在运行的时候，如何判断当前的系统版本？

可以使用 Environment 来判断，`Environment.Is64BitProcess` 可以判断当前的程序运行的是 32 还是 64 ，通过 `Environment.Is64BitOperatingSystem` 可以判断系统

那么如何判断一个 dll 是 x86 还是 any cpu？

可以打开开发者命令，然后输入 corflags 判断，开发者命令一般如果是安装 vs2017 就是`Developer Command Prompt for VS 2017`，通过开始就可以找到。

```csharp
corflags lindexi.dll
```

看输出，就可以知道这个库是什么

Any CPU:

PE: PE32
32BIT: 0

x86:

PE: PE32
32BIT: 1

x64:

PE: PE32+
32BIT: 0

除了上面几个之外，还有其他的编译选择，请看下面

- anycpu 默认的编译

- anycpu32bitpreferred 在  .NET Framework 4.5 和以上才可以使用

- ARM 程序编译为 ARM 运行

- x64

- x86

- Itanium 

如果使用命令编译，那么可以使用`platform`和字符串

```csharp
csc /platform:anycpu filename.cs  
```

参见：https://stackoverflow.com/a/12066861/6116637

[使 32 位程序使用大于 2GB 的内存 - walterlv](https://walterlv.github.io/windows/2017/09/12/32bit-application-use-large-memory.html )

如果发现引用了 dll 出现了下面的错误

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20181917348.jpg)

那么就需要检查是不是软件的环境和 dll 的环境不一样，如 软件是 x86 dll 是 x64就会出现这个问题。

如果 dll 不是 .net 的，那么可以使用下面的代码查看他环境

```csharp
dumpbin /headers foo.dll
```



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  