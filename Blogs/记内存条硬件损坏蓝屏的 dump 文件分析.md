本文记录我的电脑的内存条的硬件损坏了，导致用着用着就蓝屏，我通过启动和故障恢复配置自动蓝屏打 dump 的功能，在蓝屏时创建了 dump 文件。通过分析 dump 文件大概猜测是内存的问题

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

以下是我用 windbg 分析的两次蓝屏

第一次是用着 VS 的过程蓝屏了，我发现好多次都是使用 VS 蓝屏的，一开始还以为是 VS 投毒了

```
PROCESS_NAME:  devenv.exe

TRAP_FRAME:  ffff9f8e53267420 -- (.trap 0xffff9f8e53267420)
NOTE: The trap frame does not contain all registers.
Some register values may be zeroed or incorrect.
rax=0000000000000000 rbx=0000000000000000 rcx=000010e800000006
rdx=d70dab047be00460 rsi=0000000000000000 rdi=0000000000000000
rip=fffff8054bd520ed rsp=ffff9f8e532675b0 rbp=ffff9f8e53267820
 r8=00007ff8221127be  r9=ffffa0800f9ad7f0 r10=8040000000201039
r11=ffffadd6eb75b7f8 r12=0000000000000000 r13=0000000000000000
r14=0000000000000000 r15=0000000000000000
iopl=0         nv up ei ng nz na pe nc
nt!MiHandleTransitionFault+0xed:
fffff805`4bd520ed 8b4138          mov     eax,dword ptr [rcx+38h] ds:000010e8`0000003e=????????
Resetting default scope

STACK_TEXT:  
ffff9f8e`532672d8 fffff805`4be2be29 : 00000000`0000000a 000010e8`0000003e 00000000`00000002 00000000`00000000 : nt!KeBugCheckEx
ffff9f8e`532672e0 fffff805`4be27289 : 0000007f`00000000 00007ff8`219ef000 00000000`00000008 03a5ce00`00000000 : nt!KiBugCheckDispatch+0x69
ffff9f8e`53267420 fffff805`4bd520ed : ffff9f8e`53267a20 1a000004`38454867 ffffa080`0d581a20 fffff805`4bccdadc : nt!KiPageFault+0x489
ffff9f8e`532675b0 fffff805`4bd517d6 : ffff9f8e`532677f8 ffffb009`d4f2cc90 ffffa080`06030ab0 00000000`00000000 : nt!MiHandleTransitionFault+0xed
ffff9f8e`53267630 fffff805`4bd51113 : ffff9f8e`53267820 ffffadd6`eb6fff00 ffffa080`06030ab0 00000000`00000000 : nt!MiResolveTransitionFault+0x156
ffff9f8e`532676f0 fffff805`4bd4abea : ffff9f8e`53267820 00000000`00000000 ffff9f8e`532677f8 00000000`00000000 : nt!MiResolveProtoPteFault+0x7a3
ffff9f8e`532677c0 fffff805`4bd47db2 : ffffd70d`b1b80700 00000000`00000000 00000000`c0000016 00000034`00000000 : nt!MiDispatchFault+0x3ca
ffff9f8e`53267900 fffff805`4be2717e : ffffd70d`ab42b080 00000154`0fe40000 00000000`020004db 00000000`00000004 : nt!MmAccessFault+0x152
ffff9f8e`53267a20 00007ff8`acf1f71d : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : nt!KiPageFault+0x37e
00000034`ab1aa150 00000000`00000000 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : 0x00007ff8`acf1f71d
```

通过以上的 `nt!KiPageFault` 函数，即内存缺页错误堆栈，大概可以猜测和内存相关

第二个 dump 分析如下

```
PROCESS_NAME:  svchost.exe

TRAP_FRAME:  ffff8b82b28b6d60 -- (.trap 0xffff8b82b28b6d60)
NOTE: The trap frame does not contain all registers.
Some register values may be zeroed or incorrect.
rax=9c8b7ed481500420 rbx=0000000000000000 rcx=0000000000000000
rdx=80400000001a47a8 rsi=0000000000000000 rdi=0000000000000000
rip=fffff8051973eaf7 rsp=ffff8b82b28b6ef0 rbp=ffff8b82b28b6f89
 r8=0a00000000000060  r9=0000000000000000 r10=0000000000000006
r11=47a8000000030546 r12=0000000000000000 r13=0000000000000000
r14=0000000000000000 r15=0000000000000000
iopl=0         nv up ei pl zr na po nc
nt!MiIdentifyPfn+0x5a7:
fffff805`1973eaf7 f00fba6e481f    lock bts dword ptr [rsi+48h],1Fh ds:00000000`00000048=????????
Resetting default scope

STACK_TEXT:  
ffff8b82`b28b6c18 fffff805`1982be29 : 00000000`0000000a 00000000`00000048 00000000`00000002 00000000`00000001 : nt!KeBugCheckEx
ffff8b82`b28b6c20 fffff805`19827289 : ffff8b82`0000018e fffff805`19735df5 ffff8b82`b28b6d90 ffff8b82`b28b6e08 : nt!KiBugCheckDispatch+0x69
ffff8b82`b28b6d60 fffff805`1973eaf7 : 00000000`000018c0 00000000`00000000 00000000`42506650 fffff805`196bf5af : nt!KiPageFault+0x489
ffff8b82`b28b6ef0 fffff805`1973c915 : 00000000`00000100 00000026`94ffa100 00000000`00000000 fffff805`19ba44c5 : nt!MiIdentifyPfn+0x5a7
ffff8b82`b28b6ff0 fffff805`19be48ca : ffff9c8b`00000000 ffff9c8b`7cfa6a50 ffff9c8b`7cf76080 ffff9c8b`7cfa78c0 : nt!MiIdentifyPfnWrapper+0x75
ffff8b82`b28b7030 fffff805`19ba41b4 : 00000000`00000000 00000000`00000001 ffff9c8b`7cfa6000 00000000`00000001 : nt!MmQueryPfnList+0x7a
ffff8b82`b28b7070 fffff805`19ba3deb : 00000000`00000000 00000000`00000000 ffffc000`08db9730 ffff9c8b`7cfa6000 : nt!PfpPfnPrioRequest+0x80
ffff8b82`b28b70e0 fffff805`19ba1dfd : 00000026`94ffa0a8 fffff805`196c3657 00000000`0000004f 00000000`00000000 : nt!PfQuerySuperfetchInformation+0xe7
ffff8b82`b28b71b0 fffff805`19ba1aad : 00000000`00000000 00000026`94ffd300 00000026`94ffa0e0 00000000`00000000 : nt!ExpQuerySystemInformation+0x2ed
ffff8b82`b28b79e0 fffff805`1982b505 : 00000026`94ff0000 00000000`00000000 00000000`00000008 00000000`00000a0a : nt!NtQuerySystemInformation+0x5d
ffff8b82`b28b7a20 00007ffc`38070664 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : nt!KiSystemServiceCopyEnd+0x25
00000026`94ff9fa8 00000000`00000000 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : 0x00007ffc`38070664
```

也是挂在 KiPageFault 这里，证明应该就是内存相关问题。看到以上堆栈，只可以猜测是内存相关问题，不知道是驱动层问题，还是超频问题，还是内存条硬件损坏问题

在 [lsj](https://blog.sdlsj.net) 的协助下，我使用了 [TestMem5](https://test-mem-5.com/) 工具进行测试，测试到了是一条内存条硬件损坏

拆掉损坏的内存条就不会蓝屏了

以上的蓝屏 dump 文件是通过在高级系统配置里面，点击启动和故障恢复里，配置系统失败时写入调试信息

<!-- ![](image/记内存条硬件损坏蓝屏的 dump 文件分析/记内存条硬件损坏蓝屏的 dump 文件分析0.png) -->
![](http://image.acmx.xyz/lindexi%2F202479142210833.jpg)

在此之前我使用 Windows 自带的 `控制面板\系统和安全\Windows 工具` 的 Windows 内存诊断 工具都扫描不出来问题，这是因为此工具的内存压力不够大

这也就是为什么经常炸在 VS 的原因，因为 VS 给内存的压力足够大。使用 [TestMem5](https://test-mem-5.com/) 工具进行测试能够更好找到坑
