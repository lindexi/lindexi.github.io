# 记腾讯微信输入法导致 WPF 应用卡住 窗口无法激活和显示

本文记录我远程一位老师的设备，这位老师使用的是 Win7 Sp1 系统，现象是 WPF 应用的窗口无法激活，在 user32 的 SetForegroundWindow 或 NtUserShowWindow 方法卡住

<!--more-->
<!-- CreateTime:2024/11/21 07:10:00 -->

<!-- 发布 -->
<!-- 博客 -->

我收集到了多个进程的卡住的 dump 文件，分析到卡住有两个不同的堆栈

堆栈1：

```
 	user32.dll!_NtUserShowWindow@8()	未知
 	[托管到本机的转换]	
>	PresentationFramework.dll!System.Windows.Window.ShowHelper(object booleanBox = false) 行 3293	C#
 	PresentationFramework.dll!System.Windows.Window.Hide() 行 1358	C#
    业务代码
```

堆栈2：

```
>	user32.dll!_NtUserCallHwndLock@8()	未知
 	user32.dll!_SetForegroundWindow@4()	未知
 	[托管到本机的转换]	
 	PresentationFramework.dll!System.Windows.Window.Activate() 行 1467	C#
 	业务代码
```

从进程模块上可以看到有微信输入法的以下模块注入：

- C:\Program Files\Tencent\WeType\WetypeCore_1.0.4.289\x86\CrashRpt1500.dll 1.05.0.0
- C:\Program Files\Tencent\WeType\WetypeCore_1.0.4.289\x86\dbghelp.dll 10.0.10150.0(debuggers(- dbg).150616-1659)
- C:\Program Files\Tencent\WeType\WetypeCore_1.0.4.289\x86\wetype_tip_core.dll 1.00.4.289
- C:\Program Files\Tencent\WeType\WetypeCore_1.0.4.289\x86\WeUIResource.dll 1.00.0.0

复现步骤：

反复进入可能会激活显示窗口和隐藏窗口的逻辑即可

解决方法：

卸载腾讯微信输入法

卸载完成之后，重启进程即可修复

相关问题： [记微信截图导致 WPF 应用卡住 窗口无法激活问题](https://blog.lindexi.com/post/%E8%AE%B0%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE%E5%AF%BC%E8%87%B4-WPF-%E5%BA%94%E7%94%A8%E5%8D%A1%E4%BD%8F-%E7%AA%97%E5%8F%A3%E6%97%A0%E6%B3%95%E6%BF%80%E6%B4%BB%E9%97%AE%E9%A2%98.html )

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )