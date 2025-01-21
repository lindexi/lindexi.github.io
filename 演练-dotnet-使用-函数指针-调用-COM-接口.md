
# 演练 dotnet 使用 函数指针 调用 COM 接口

本文将和大家演练如何在 dotnet 里面使用 函数指针 调用 COM 接口，整个过程没有 COM 封装的存在，其性能非常高，调用非常直接和底层，无中间商赚差价

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

官方文档：

[函数指针 - C# feature specifications Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/proposals/csharp-9.0/function-pointers )

[ComWrappers source generation - .NET Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/comwrappers-source-generation )

[Using the ComWrappers API - .NET Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/tutorial-comwrappers )

在 .NET 5 之前，绝大部分的 COM 调用逻辑都是在运行时生成封送辅助对象，即 .NET COM Wrapper 对象。这个过程中，对 COM 的访问是需要有 .NET 中间商对象，中间商对象的调用是有损耗的。对于某些调用逻辑来说，这些损耗可能占比比较多

在 C# 9 时，引入了函数指针（Function pointers）的概念，通过此可以直接指针方式调用 COM 接口，提供非常高的性能，调用方式非常直接和底层

本文接下来将演练调用 COM 接口，使用 IFileOpenDialog 启动打开文件对话框

本文代码项目无三方依赖，完全在一个 Program.cs 文件完成

本文内容里面只给出关键代码片段，如需要全部的项目文件，可到本文末尾找到本文所有代码的下载方法

先定义两个必要的 Win32 方法，代码如下

```csharp
[DllImport("OLE32.dll", ExactSpelling = true, EntryPoint = "CoInitializeEx")]
static extern unsafe int CoInitializeEx([Optional] void* pvReserved, [MarshalAs(UnmanagedType.U4)] COINIT dwCoInit);

[DllImport("OLE32.dll", ExactSpelling = true)]
[DefaultDllImportSearchPaths(DllImportSearchPath.System32)]
static extern unsafe int CoCreateInstance(Guid* rclsid, IntPtr pUnkOuter, CLSCTX dwClsContext, Guid* riid, void* ppv);
```

我这里新建的是控制台项目，需要调用 CoInitializeEx 进行初始化

```csharp
    CoInitializeEx(null, COINIT.COINIT_APARTMENTTHREADED | COINIT.COINIT_DISABLE_OLE1DDE);
```

接着使用 CoCreateInstance 拿到 IFileOpenDialog 的 COM 指针

```csharp
    var rclsid = Guid.Parse("DC1C5A9C-E88A-4dde-A5A1-60F82A20AEF7");
    var riid = Guid.Parse("d57c7288-d4ad-4768-be02-9d969532d960");

    void* ppv;

    var hr = CoCreateInstance(&rclsid, IntPtr.Zero,
        CLSCTX.CLSCTX_ALL, &riid, &ppv);
    Debug.Assert(hr == 0);
```

此时可通过 ppv 获取方法表，代码如下

```csharp
    void*** pvtable = (void***) ppv;
```

再通过 IFileOpenDialog 的 `void Show(IntPtr owner);` 方法定义即可写出函数指针调用

```csharp
    void*** pvtable = (void***) ppv;
    // void Show(IntPtr owner);
    var owner = IntPtr.Zero;
    ((delegate* unmanaged[MemberFunction]<void*, nint, int>) (*pvtable)[3])(ppv, owner);
```

在 COM 调用对应的函数指针里面，首个参数是 ppv 指针，其次的 `nint` 是 `void Show(IntPtr owner);` 方法的 `IntPtr owner` 参数。最后一个 int 是返回值

由于第 0 1 2 的函数是 IUnknown 的 AddRef Release QueryInterface 默认函数。因此一般来说都是从第 3 个开始才是实际接口的方法。于是从 `(*pvtable)[3]` 就可以取出 COM 对应的方法指针。再将其转换为 `(delegate* unmanaged[MemberFunction]<void*, nint, int>)` 函数指针即可进行调用

尝试运行代码，可见此时弹出文件对话框

本文例子有很多不足点，如没有开启消息循环，没有合理处理文件对话框后续逻辑以及完成 COM 释放。本文只是和大家演示如何使用函数指针调用 COM 接口

当前许许多多新的 .NET 组件库，在选择调用 COM 接口时，都会优先采用函数指针的方式。如 Silk.NET 库等等。这些基础库通过函数指针调用的方式，可以做到让 .NET 调用 COM 时，无需中间商赚差价，在 .NET 层的调用损耗几乎可以忽略不计。本文介绍的此函数指针调用方式也适用于 WinRT 接口

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/6a5e8a6016cb47b98959a6a4261cef400e0a5bcc/Workbench/HaiwahalheewawWaychaiderekebe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6a5e8a6016cb47b98959a6a4261cef400e0a5bcc/Workbench/HaiwahalheewawWaychaiderekebe) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6a5e8a6016cb47b98959a6a4261cef400e0a5bcc
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6a5e8a6016cb47b98959a6a4261cef400e0a5bcc
```

获取代码之后，进入 Workbench/HaiwahalheewawWaychaiderekebe 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。