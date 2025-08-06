---
title: 记 Kylin 麒麟系统安全中心拦截导致 dotnet sdk 找不到 OpenSsl 构建失败
description: 我在龙芯机器上安装了 Kylin 麒麟系统，然后去下载龙芯的 dotnet sdk 安装，接着尝试构建一个简单应用，却发现构建失败。报错是 System.DllNotFoundException: Unable to load shared library 'libSystem.Security.Cryptography.Native.OpenSsl' or one of its dependencies
tags: dotnet
category: 
---

<!-- CreateTime:2024/10/30 07:09:52 -->

<!-- 发布 -->
<!-- 博客 -->

错误信息如下

```
System.TypeInitializationException: The type initializer for 'Crypto' threw an exception.
 ---> System.DllNotFoundException: Unable to load shared library 'libSystem.Security.Cryptography.Native.OpenSsl' or one of its dependencies. In order to help diagnose loading problems, consider using a tool like strace. If you're using glibc, consider setting the LD_DEBUG environment variable:
/home/lindexi/wzc/dotnet/shared/Microsoft.NETCore.App/8.0.7/libSystem.Security.Cryptography.Native.OpenSsl.so: failed to map segment from shared object
libSystem.Security.Cryptography.Native.OpenSsl.so: cannot open shared object file: No such file or directory
/home/lindexi/wzc/dotnet/shared/Microsoft.NETCore.App/8.0.7/liblibSystem.Security.Cryptography.Native.OpenSsl.so: cannot open shared object file: No such file or directory
/home/lindexi/wzc/dotnet/shared/Microsoft.NETCore.App/8.0.7/libSystem.Security.Cryptography.Native.OpenSsl: cannot open shared object file: No such file or directory
/home/lindexi/wzc/dotnet/shared/Microsoft.NETCore.App/8.0.7/liblibSystem.Security.Cryptography.Native.OpenSsl: cannot open shared object file: No such file or directory

   at Interop.Crypto..cctor()
   --- End of inner exception stack trace ---
   at Interop.Crypto.HashAlgorithmToEvp(String hashAlgorithmId)
   at System.Security.Cryptography.HashProviderDispenser.OneShotHashProvider.HashData(String hashAlgorithmId, ReadOnlySpan`1 source, Span`1 destination)
   at System.Security.Cryptography.SHA256.TryHashData(ReadOnlySpan`1 source, Span`1 destination, Int32& bytesWritten)
   at System.Security.Cryptography.SHA256.HashData(ReadOnlySpan`1 source, Span`1 destination)
   at System.Security.Cryptography.SHA256.HashData(ReadOnlySpan`1 source)
   at System.Security.Cryptography.SHA256.HashData(Byte[] source)
   at Microsoft.DotNet.Cli.Utils.Sha256Hasher.Hash(String text)
   at Microsoft.DotNet.Cli.Utils.Sha256Hasher.HashWithNormalizedCasing(String text)
   at Microsoft.DotNet.Cli.Utils.ApplicationInsightsEntryFormat.<>c__DisplayClass10_0.<WithAppliedToPropertiesValue>b__1(KeyValuePair`2 p)
   at System.Linq.Enumerable.ToDictionary[TSource,TKey,TElement](IEnumerable`1 source, Func`2 keySelector, Func`2 elementSelector, IEqualityComparer`1 comparer)
   at System.Linq.Enumerable.ToDictionary[TSource,TKey,TElement](IEnumerable`1 source, Func`2 keySelector, Func`2 elementSelector)
   at Microsoft.DotNet.Cli.Utils.ApplicationInsightsEntryFormat.WithAppliedToPropertiesValue(Func`2 func)
   at Microsoft.DotNet.Cli.Telemetry.TelemetryFilter.<Filter>b__3_0(ApplicationInsightsEntryFormat r)
   at System.Linq.Enumerable.SelectListIterator`2.Fill(ReadOnlySpan`1 source, Span`1 destination, Func`2 func)
   at System.Linq.Enumerable.SelectListIterator`2.ToList()
   at Microsoft.DotNet.Cli.Telemetry.TelemetryFilter.Filter(Object objectToFilter)
   at Microsoft.DotNet.Cli.Utils.TelemetryEventEntry.SendFiltered(Object o)
   at Microsoft.DotNet.Cli.Program.ProcessArgs(String[] args, TimeSpan startupTime, ITelemetry telemetryClient)
   at Microsoft.DotNet.Cli.Program.Main(String[] args)
```

尝试设置 `export LD_DEBUG=all` 命令寻找依赖，却没有看到有用的信息，能看到寻找信息如下

```
binding file /lib/loongarch64-linux-gnu/libssl.so.1.1 [0] to /lib/loongarch64-linux-gnu/libcrypto.so.1.1 [0]: normal symbol `UINT32_it' [OPENSSL_1_1_0f]
```

且能够在机器上找到此文件

```
/lib/loongarch64-linux-gnu$ ls | grep libssl
libssl3.so
libssl.so.1.1
```

尝试使用 `LD_LIBRARY_PATH` 环境变量设置寻找路径，依然没有帮助，依然提示上述错误

尝试重新去龙芯官方下载 dotnet sdk 安装，也没有解决问题，下载地址： <http://ftp.loongnix.cn/dotnet/8.0.7/8.0.7-1/deb/dotnet-sdk-8.0_8.0.107-1_loongarch64.deb>

我甚至都开始怀疑是龙芯[新旧世界](https://areweloongyet.com/docs/old-and-new-worlds/)的问题了，因为龙芯提供的 dotnet 是旧世界的，我跑的麒麟系统也是旧世界的。通过安同的文档可知，只需判断 `/lib64` 路径下是否只有 `ld.so.1` 文件即可知道是否旧世界的系统

```
/lib64$ ls
ld.so.1
```

再使用 file 命令查看 dotnet 入口程序文件

```
file dotnet
dotnet: ELF 64-bit LSB shared object, LoongArch-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld.so.1, for GNU/Linux 4.15.0, BuildID[sha1]=b1631460420e1fb663d140cc85a9b39b7783f7f3, stripped
```

通过以上命令的 `interpreter /lib64/ld.so.1` 可以知道龙芯提供的 dotnet sdk 也是旧世界的。新世界的 dotnet 是在龙芯社区组织里面的，详细请看 <https://github.com/loongson-community/dotnet-unofficial-build>

以上这些推测都不正确，我后面在调试别的问题的时候，发现了麒麟系统的安全中心才发现了核心问题

核心原因是我是使用 ssh 远程过去的，麒麟系统的安全中心将我的 dotnet sdk 运行给拦截了，但是在 SSH 里面啥都没有提示，啥都没有看见

解决方法就是在实体机器上，插入显示器和键盘鼠标，再敲一次 dotnet build 命令，接着将弹出的安全中心的未授权程序都点允许

点击那会我忘记截图了，下图可见是在麒麟系统的通知栏上的内容

<!-- ![](image/记 Kylin 麒麟系统安全中心拦截导致 dotnet sdk 找不到 OpenSsl 构建失败/记 Kylin 麒麟系统安全中心拦截导致 dotnet sdk 找不到 OpenSsl 构建失败0.png) -->
![](http://cdn.lindexi.site/lindexi%2F202410291458337190.jpg)

点击允许之后，再次在远程的 ssh 里面执行 dotnet build 就都能通过了

如此证明龙芯提供的 dotnet sdk 是没有问题的。只是麒麟系统的安全中心在逗我

参考文档：

- [Unable to load shared library 'libFabricCommon.so' or one of its dependencies. · Issue #1203 · microsoft/service-fabric-issues](https://github.com/microsoft/service-fabric-issues/issues/1203 )
- <https://github.com/loongson-community/dotnet-unofficial-build>
- [新旧世界](https://areweloongyet.com/docs/old-and-new-worlds/)
- <http://ftp.loongnix.cn/dotnet/8.0.7/8.0.7-1/deb/dotnet-sdk-8.0_8.0.107-1_loongarch64.deb>
- [我需要帮助，关于gcc的 - LA UOSC](https://bbs.loongarch.org/d/224-gcc )
- [The program cannot be started on Linux , Loongson, Arch · Issue #7747 · AvaloniaUI/Avalonia](https://github.com/AvaloniaUI/Avalonia/issues/7747 )
- <https://github.com/shushanhf/runtime>
- [Add LoongArch64 architecture port · Issue #59561 · dotnet/runtime](https://github.com/dotnet/runtime/issues/59561 )
- <https://nuget.loongnix.cn/packages/Microsoft.NETCore.App.Runtime.linux-loongarch64>
- [LoongArch & Avalonia](https://avaloniaui.net/blog/loongarch-avalonia )
- [常见问题(FAQ) · 文档](https://docs.loongnix.cn/dotnet/support/list/01.%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98-FAQ.html )
- [在LoongArch Loongnix开发C#的问题 - LA UOSC](https://bbs.loongarch.org/d/266-loongarch-loongnixc )
- [The unofficial yet comprehensive FAQ for LoongArch (last updated 2022-11-23) write(2)](https://blog.xen0n.name/en/posts/tinkering/loongarch-faq/ )
- [[LoongArch64] A plan for amending the LoongArch64‘s port · Issue #69705 · dotnet/runtime](https://github.com/dotnet/runtime/issues/69705 )

其他拦截问题请看： [dotnet 记龙芯麒麟教育版安全中心拦截文件 导致 docker 内 CI CD 构建失败](https://blog.lindexi.com/post/dotnet-%E8%AE%B0%E9%BE%99%E8%8A%AF%E9%BA%92%E9%BA%9F%E6%95%99%E8%82%B2%E7%89%88%E5%AE%89%E5%85%A8%E4%B8%AD%E5%BF%83%E6%8B%A6%E6%88%AA%E6%96%87%E4%BB%B6-%E5%AF%BC%E8%87%B4-docker-%E5%86%85-CI-CD-%E6%9E%84%E5%BB%BA%E5%A4%B1%E8%B4%A5.html )
<!-- [dotnet 记龙芯麒麟教育版安全中心拦截文件 导致 docker 内 CI CD 构建失败 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18545167 ) -->
