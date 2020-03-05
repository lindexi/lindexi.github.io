# dotnet 获取指定进程的输入命令行

本文告诉大家如何在 dotnet 获取指定的进程的命令行参数

<!--more-->
<!-- CreateTime:2019/11/29 8:35:11 -->


很多的程序在启动的时候都需要传入参数，那么如何拿到这些程序传入的参数？

我找到两个方法，一个需要引用 C++ 库支持 x86 和 x64 程序，另一个都是C#代码，但是只支持 x64 程序

本文提供一个由 [StackOverflow](https://stackoverflow.com/q/2633628/6116637) 大神开发的库拿到进程的命令行

在使用下面的代码需要引用两个 C++ 的库，可以从 [csdn](https://download.csdn.net/download/lindexi_gd/10970169 ) 下载

使用下面的代码就可以拿到传入进程的参数，在使用之前，需要在输出的文件夹里面包含 ProcCmdLine32.dll 和 ProcCmdLine64.dll 可以从[csdn](https://download.csdn.net/download/lindexi_gd/10970169 ) 下载
 
```csharp
        public static string GetCommandLineOfProcess(Process process)
        {
            // max size of a command line is USHORT/sizeof(WCHAR), so we are going
            // just allocate max USHORT for sanity sake.
            var stringBuilder = new StringBuilder(0xFFFF);
            if (Environment.Is64BitProcess)
            {
                GetProcCmdLine64((uint) process.Id, stringBuilder, (uint) stringBuilder.Capacity);
            }
            else
            {
                GetProcCmdLine32((uint) process.Id, stringBuilder, (uint) stringBuilder.Capacity);
            }

            return stringBuilder.ToString();
        }

        [DllImport("ProcCmdLine32.dll", CharSet = CharSet.Unicode, EntryPoint = "GetProcCmdLine")]
        private static extern bool GetProcCmdLine32(uint nProcId, StringBuilder stringBuilder, uint dwSizeBuf);

        [DllImport("ProcCmdLine64.dll", CharSet = CharSet.Unicode, EntryPoint = "GetProcCmdLine")]
        private static extern bool GetProcCmdLine64(uint nProcId, StringBuilder stringBuilder, uint dwSizeBuf);
```

获取所有的进程的命令行可以使用这个代码

```csharp
            foreach (var process in Process.GetProcesses())
            {
                Console.WriteLine($"{process.ProcessName} {GetCommandLineOfProcess(process)}");
            }
```

代码请看 https://github.com/lindexi/lindexi_gd/tree/cf4054b0f479986bd295a8e5b69c31ad8fd7fe10/GetProcessCommandLine

上面的代码需要引用一个 C++ 的库，看起来不清真，下面通过全部 C# 的代码

```csharp
        public static string GetCommandLineOfProcess(int processId)
        {
            var pid = processId;

            var pbi = new NativeMethods.PROCESS_BASIC_INFORMATION();

            IntPtr proc = NativeMethods.OpenProcess
            (
                PROCESS_QUERY_INFORMATION | PROCESS_VM_READ, false, pid
            );

            if (proc == IntPtr.Zero)
            {
                return "";
            }

            if (NativeMethods.NtQueryInformationProcess(proc, 0, ref pbi, pbi.Size, IntPtr.Zero) == 0)
            {
                var buff = new byte[IntPtr.Size];
                if (NativeMethods.ReadProcessMemory
                (
                    proc,
                    (IntPtr) (pbi.PebBaseAddress.ToInt32() + 0x10),
                    buff,
                    IntPtr.Size, out _
                ))
                {
                    var buffPtr = BitConverter.ToInt32(buff, 0);
                    var commandLine = new byte[Marshal.SizeOf(typeof(NativeMethods.UNICODE_STRING))];

                    if
                    (
                        NativeMethods.ReadProcessMemory
                        (
                            proc, (IntPtr) (buffPtr + 0x40),
                            commandLine,
                            Marshal.SizeOf(typeof(NativeMethods.UNICODE_STRING)), out _
                        )
                    )
                    {
                        var ucsData = ByteArrayToStructure<NativeMethods.UNICODE_STRING>(commandLine);
                        var parms = new byte[ucsData.Length];
                        if
                        (
                            NativeMethods.ReadProcessMemory
                            (
                                proc, ucsData.buffer, parms,
                                ucsData.Length, out _
                            )
                        )
                        {
                            return Encoding.Unicode.GetString(parms);
                        }
                    }
                }
            }

            NativeMethods.CloseHandle(proc);

            return "";
        }

        private const uint PROCESS_QUERY_INFORMATION = 0x400;
        private const uint PROCESS_VM_READ = 0x010;

        private static T ByteArrayToStructure<T>(byte[] bytes) where T : struct
        {
            var handle = GCHandle.Alloc(bytes, GCHandleType.Pinned);
            var stuff = (T) Marshal.PtrToStructure(handle.AddrOfPinnedObject(), typeof(T));
            handle.Free();
            return stuff;
        }

        private static class NativeMethods
        {
            [DllImport("kernel32.dll", SetLastError = true)]
            [return: MarshalAs(UnmanagedType.Bool)]
            internal static extern bool CloseHandle(IntPtr hObject);

            [DllImport("kernel32.dll", SetLastError = true)]
            internal static extern IntPtr OpenProcess
            (
                uint dwDesiredAccess,
                [MarshalAs(UnmanagedType.Bool)] bool bInheritHandle,
                int dwProcessId
            );


            [DllImport("kernel32.dll", SetLastError = true)]
            [return: MarshalAs(UnmanagedType.Bool)]
            internal static extern bool ReadProcessMemory
            (
                IntPtr hProcess,
                IntPtr lpBaseAddress,
                byte[] lpBuffer,
                int nSize,
                out IntPtr lpNumberOfBytesRead
            );

            [DllImport("ntdll.dll")]
            internal static extern int NtQueryInformationProcess
            (
                IntPtr ProcessHandle,
                uint ProcessInformationClass,
                ref PROCESS_BASIC_INFORMATION ProcessInformation,
                uint ProcessInformationLength,
                IntPtr ReturnLength
            );

            [StructLayout(LayoutKind.Sequential, Pack = 1)]
            internal struct PROCESS_BASIC_INFORMATION
            {
                internal int ExitProcess;
                internal IntPtr PebBaseAddress;
                internal IntPtr AffinityMask;
                internal int BasePriority;
                internal IntPtr UniqueProcessId;
                internal IntPtr InheritedFromUniqueProcessId;

                internal uint Size => (uint) Marshal.SizeOf(typeof(PROCESS_BASIC_INFORMATION));
            }

            [StructLayout(LayoutKind.Sequential, Pack = 1)]
            internal struct UNICODE_STRING
            {
                internal ushort Length;
                internal ushort MaximumLength;
                internal IntPtr buffer;
            }
        }
```

获取所有进程的参数

```csharp
        [STAThread]
        private static void Main(string[] args)
        {
            if (Environment.Is64BitProcess)
            {
                throw new InvalidOperationException("暂时只支持x86程序");
            }

            foreach (var process in Process.GetProcesses())
            {
                Console.WriteLine($"{process.ProcessName} {GetCommandLineOfProcess(process.Id)}");
            }
        }
```

更简单是[通过 WMI 获取指定进程的输入命令行](https://blog.lindexi.com/post/dotnet-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E6%8C%87%E5%AE%9A%E8%BF%9B%E7%A8%8B%E7%9A%84%E8%BE%93%E5%85%A5%E5%91%BD%E4%BB%A4%E8%A1%8C.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
