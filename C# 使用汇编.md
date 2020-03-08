# C# 使用汇编

本文告诉大家如何在 C# 里面使用汇编代码

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


请看

[C#嵌入x86汇编——一个GPIO接口的实现 - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1016850 )

[C# inline-asm / 嵌入x86汇编 - 苏璃 - CSDN博客](https://blog.csdn.net/u012395622/article/details/46801475 )

通过这个方法在 dotnet core 获取 CPU 信息

```csharp
[StructLayout(LayoutKind.Sequential)]
internal ref struct CpuIdInfo
{
    public uint Eax;
    public uint Ebx;
    public uint Ecx;
    public uint Edx;


    public static void AppendAsString(StringBuilder builder,uint value)
    {
        var val = value;

        while (val != 0)
        {
            builder.Append((char) (val & 0xFF));
            val >>= 8;
        }

    }

    public string GetString()
    {
        StringBuilder ret = new StringBuilder(16);
        AppendAsString(ret,Ebx);
        AppendAsString(ret,Edx);
        AppendAsString(ret,Ecx);

        return ret.ToString();
    }
}
internal sealed class CpuIdAssemblyCode
    : IDisposable
{
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    private delegate void CpuIDDelegate(int level, ref CpuIdInfo cpuId);

    private IntPtr _codePointer;
    private uint _size;
    private CpuIDDelegate _delegate;

    public CpuIdAssemblyCode()
    {
        byte[] codeBytes = (IntPtr.Size == 4) ? x86CodeBytes : x64CodeBytes;

        _size = (uint) codeBytes.Length;
        _codePointer = NativeMethods.Kernel32.VirtualAlloc(
                IntPtr.Zero,
                new UIntPtr(_size),
                AllocationType.COMMIT | AllocationType.RESERVE,
                MemoryProtection.EXECUTE_READWRITE
            );

        Marshal.Copy(codeBytes, 0, _codePointer, codeBytes.Length);
#if NET40
        _delegate = (CpuIDDelegate) Marshal.GetDelegateForFunctionPointer(_codePointer, typeof(CpuIDDelegate));
#else
        _delegate = Marshal.GetDelegateForFunctionPointer<CpuIDDelegate>(_codePointer);
#endif

    }

    ~CpuIdAssemblyCode()
    {
        Dispose(false);
    }

    public void Call(int level, ref CpuIdInfo cpuInfo)
    {
        _delegate(level, ref cpuInfo);
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    private void Dispose(bool disposing)
    {
        NativeMethods.Kernel32.VirtualFree(_codePointer, _size, 0x8000);
    }

    // Basic ASM strategy --
    // void x86CpuId(int level, byte* buffer) 
    // {
    //    eax = level
    //    cpuid
    //    buffer[0] = eax
    //    buffer[4] = ebx
    //    buffer[8] = ecx
    //    buffer[12] = edx
    // }

    private readonly static byte[] x86CodeBytes = 
    {
    0x55,                   // push        ebp  
    0x8B, 0xEC,             // mov         ebp,esp
    0x53,                   // push        ebx  
    0x57,                   // push        edi

    0x8B, 0x45, 0x08,       // mov         eax, dword ptr [ebp+8] (move level into eax)
    0x0F, 0xA2,              // cpuid

    0x8B, 0x7D, 0x0C,       // mov         edi, dword ptr [ebp+12] (move address of buffer into edi)
    0x89, 0x07,             // mov         dword ptr [edi+0], eax  (write eax, ... to buffer)
    0x89, 0x5F, 0x04,       // mov         dword ptr [edi+4], ebx 
    0x89, 0x4F, 0x08,       // mov         dword ptr [edi+8], ecx 
    0x89, 0x57, 0x0C,       // mov         dword ptr [edi+12],edx 

    0x5F,                   // pop         edi  
    0x5B,                   // pop         ebx  
    0x8B, 0xE5,             // mov         esp,ebp  
    0x5D,                   // pop         ebp 
    0xc3                    // ret
    };

    private readonly static byte[] x64CodeBytes = 
    {
    0x53,                       // push rbx    this gets clobbered by cpuid

    // rcx is level
    // rdx is buffer.
    // Need to save buffer elsewhere, cpuid overwrites rdx
    // Put buffer in r8, use r8 to reference buffer later.

    // Save rdx (buffer addy) to r8
    0x49, 0x89, 0xd0,           // mov r8,  rdx

    // Move ecx (level) to eax to call cpuid, call cpuid
    0x89, 0xc8,                 // mov eax, ecx
    0x0F, 0xA2,                 // cpuid

    // Write eax et al to buffer
    0x41, 0x89, 0x40, 0x00,     // mov    dword ptr [r8+0],  eax
    0x41, 0x89, 0x58, 0x04,     // mov    dword ptr [r8+4],  ebx
    0x41, 0x89, 0x48, 0x08,     // mov    dword ptr [r8+8],  ecx
    0x41, 0x89, 0x50, 0x0c,     // mov    dword ptr [r8+12], edx

    0x5b,                       // pop rbx
    0xc3                        // ret
    };


}
```

使用方法

```csharp
var asmCode = new CpuIdAssemblyCode();
CpuIdInfo info = new CpuIdInfo();
asmCode.Call(0, ref info);
asmCode.Dispose();
string ret= info.GetString();
```

[c# - How can I get CPU name in .NET Core? - Stack Overflow](https://stackoverflow.com/questions/47254573/how-can-i-get-cpu-name-in-net-core?rq=1 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
