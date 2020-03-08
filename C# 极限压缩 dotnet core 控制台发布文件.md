# C# 极限压缩 dotnet core 控制台发布文件

每次发布 dotnet core 应用都会觉得发布文件太大，而如果使用极限压缩，用 CoreRT 能让发布的控制台文件到 5KB 左右，不需要带框架就能在其他设备运行

<!--more-->
<!-- CreateTime:2019/10/4 14:59:36 -->

<!-- csdn -->

这是微软 [MichalStrehovsky](https://github.com/MichalStrehovsky) 大佬，也就是 CoreRT 项目开发者给的方法

在开始写代码之前，需要定义一些基础的类，因为不包含任何运行环境，所以基础的 object 这些都需要重新定义，这里的代码放在 [github](https://github.com/MichalStrehovsky/zerosharp/blob/master/no-runtime/zerosharp.cs) 我也在本文最后贴出代码

现在输出控制台的代码不是原先的 Console.WriteLine 而是通过底层方法

```csharp
unsafe class Program
{
    [DllImport("api-ms-win-core-processenvironment-l1-1-0")]
    static extern IntPtr GetStdHandle(int nStdHandle);

    [DllImport("api-ms-win-core-console-l1-1-0")]
    static extern IntPtr WriteConsoleW(IntPtr hConsole, void* lpBuffer, int charsToWrite, out int charsWritten, void* reserved);

    static int Main()
    {
        string hello = "Hello world!";
        fixed (char* c = hello)
        {
            int charsWritten;
            WriteConsoleW(GetStdHandle(-11), c, hello.Length, out charsWritten, null);
        }

        return 42;
    }
}
```

最难的是如何编译这个文件

编译需要先使用 csc 编译为 IL 代码，然后通过 ilcompiler 编译为obj文件，然后通过 link 编译为运行文件

从开始菜单找到 x64 Native Tools Command Prompt for VS 2019 然后进入上面代码所在文件夹，执行下面代码编译

```csharp
csc /debug:embedded /noconfig /nostdlib /runtimemetadataversion:v4.0.30319 zerosharp.cs /out:zerosharp.ilexe /langversion:latest /unsafe
```

编译完成可以看到 zerosharp.ilexe 文件，然后通过 ilcompiler 将这个文件编译为 zerosharp.map 和 zerosharp.obj 文件

在自己的 NuGet 缓存文件里面找到 runtime.win-x64.microsoft.dotnet.ilcompiler 库，可以在资源管理器地址输入下面代码找到缓存文件

```csharp
%appdata%..\..\..\.nuget\packages\runtime.win-x64.microsoft.dotnet.ilcompiler
```

找到里面的最新版本，在文件夹里面的 tools 文件夹可以找到 ilc.exe 文件，如在我电脑的的文件是

```csharp
c:\Users\lindexi\.nuget\packages\runtime.win-x64.microsoft.dotnet.ilcompiler\1.0.0-alpha-27606-05\tools\ilc.exe
```

记下这个路径，接下来将使用这个工具编译

```csharp
>c:\Users\lindexi\.nuget\packages\runtime.win-x64.microsoft.dotnet.ilcompiler\1.0.0-alpha-27606-05\tools\ilc.exe zerosharp.ilexe -o zerosharp.obj --systemmodule zerosharp --map zerosharp.map -O
```

然后用 link 连接

```csharp
link /subsystem:console zerosharp.obj /entry:__managed__Main kernel32.lib /merge:.modules=.pdata /incremental:no
```

执行上面代码就可以编译 zerosharp.exe 文件，这个文件只有5KB可以将这个程序放在其他设备运行

下面是所有代码

```csharp
using System;
using System.Runtime.InteropServices;

#region A couple very basic things
namespace System
{
    public class Object { IntPtr m_pEEType; }
    public struct Void { }
    public struct Boolean { }
    public struct Char { }
    public struct SByte { }
    public struct Byte { }
    public struct Int16 { }
    public struct UInt16 { }
    public struct Int32 { }
    public struct UInt32 { }
    public struct Int64 { }
    public struct UInt64 { }
    public struct IntPtr { }
    public struct UIntPtr { }
    public struct Single { }
    public struct Double { }
    public abstract class ValueType { }
    public abstract class Enum : ValueType { }
    public struct Nullable<T> where T : struct { }
    
    public sealed class String { public readonly int Length; }
    public abstract class Array { }
    public abstract class Delegate { }
    public abstract class MulticastDelegate : Delegate { }

    public struct RuntimeTypeHandle { }
    public struct RuntimeMethodHandle { }
    public struct RuntimeFieldHandle { }

    public class Attribute { }

    namespace Runtime.CompilerServices
    {
        public class RuntimeHelpers
        {
            public static unsafe int OffsetToStringData => sizeof(IntPtr) + sizeof(int);
        }
    }
}
namespace System.Runtime.InteropServices
{
    public sealed class DllImportAttribute : Attribute
    {
        public DllImportAttribute(string dllName) { }
    }
}
#endregion

#region Things needed by ILC
namespace System
{
    namespace Runtime
    {
        internal sealed class RuntimeExportAttribute : Attribute
        {
            public RuntimeExportAttribute(string entry) { }
        }
    }

    class Array<T> : Array { }
}

namespace Internal.Runtime.CompilerHelpers
{
    using System.Runtime;

    class StartupCodeHelpers
    {
        [RuntimeExport("RhpReversePInvoke2")]
        static void RhpReversePInvoke2() { }
        [RuntimeExport("RhpReversePInvokeReturn2")]
        static void RhpReversePInvokeReturn2() { }
        [System.Runtime.RuntimeExport("__fail_fast")]
        static void FailFast() { while (true) ; }
        [System.Runtime.RuntimeExport("RhpPInvoke")]
        static void RphPinvoke() { }
        [System.Runtime.RuntimeExport("RhpPInvokeReturn")]
        static void RphPinvokeReturn() { }
    }
}
#endregion

unsafe class Program
{
    [DllImport("api-ms-win-core-processenvironment-l1-1-0")]
    static extern IntPtr GetStdHandle(int nStdHandle);

    [DllImport("api-ms-win-core-console-l1-1-0")]
    static extern IntPtr WriteConsoleW(IntPtr hConsole, void* lpBuffer, int charsToWrite, out int charsWritten, void* reserved);

    static int Main()
    {
        string hello = "Hello world!";
        fixed (char* c = hello)
        {
            int charsWritten;
            WriteConsoleW(GetStdHandle(-11), c, hello.Length, out charsWritten, null);
        }

        return 42;
    }
}
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
