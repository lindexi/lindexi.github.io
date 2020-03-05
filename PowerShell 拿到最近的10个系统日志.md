# PowerShell 拿到最近的10个系统日志

我最近发现我的程序总是调用一些不清真的代码，于是在运行的时候就退出了，我想要拿到系统的日志知道我的程序是怎么退出的，我如何通过 PowerShell 拿到最近的10个系统日志。为什么需要拿到最新10个日志，因为在我程序退出的时候可能也有其他的几个程序也退出了，我的输入又很慢，所以我就需要这样写

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


在 PowerShell 只需要一条命令就可以拿到最近的 10 个系统日志里面的应用程序日志

```csharp
Get-EventLog application -newest 10 | Format-List EventID,EntryType,TimeGenerated,Message,Source
```

运行的时候大概的输出是这样

```csharp
EventID       : 1001
EntryType     : Information
TimeGenerated : 2019/4/29 10:38:55
Message       : 故障存储段 125730739576，类型 5
                事件名称: PerfWatsonVS12Data
                响应: 不可用
                Cab ID: 2188124701481020576

                问题签名:
                P1: PerfWatsonTcdb
                P2: 0
                P3: 0
                P4: 0
                P5: 0
                P6:
                P7:
                P8:
                P9:
                P10:

                附加文件:
                \\?\C:\Users\lindexi\AppData\Local\Temp\VSTelem.Out\201904290238_D16.0_16.0.28729.10_38916_d38075ae-3d5a-
                45dd-818b-60761d49f451.tcdb
                \\?\C:\ProgramData\Microsoft\Windows\WER\Temp\WER5BDD.tmp.WERInternalMetadata.xml
                \\?\C:\ProgramData\Microsoft\Windows\WER\Temp\WER5C3C.tmp.xml
                \\?\C:\ProgramData\Microsoft\Windows\WER\Temp\WER5C6A.tmp.csv
                \\?\C:\ProgramData\Microsoft\Windows\WER\Temp\WER5C9A.tmp.txt
                \\?\C:\Users\lindexi\AppData\Local\Temp\WER84F3.tmp.WERDataCollectionStatus.txt

                可在此处获取这些文件:
                \\?\C:\ProgramData\Microsoft\Windows\WER\ReportArchive\NonCritical_PerfWatsonTcdb_688f135dd185f2a8133a4
                74a2518efc6ce6cc4a_00000000_cab_873deabf

                分析符号:
                重新检查解决方案: 0
                原因: 林德熙太逗比
                报告 ID: 1cbd43b1-3f84-4940-8acd-4f2bf79599ba
                报告状态: 268435464
                哈希存储段: fb30d2b9a7acf0305d4da51668259751nCab GUID: 3760cd78-5646-4c0e-ae5d-c7d2a408aca0
Source        : Windows Error Reporting
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
