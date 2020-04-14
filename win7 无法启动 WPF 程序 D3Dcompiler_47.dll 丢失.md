# win7 无法启动 WPF 程序 D3Dcompiler_47.dll 丢失

本文记录 D3Dcompiler_47 丢失问题，在安装 KB4040973 KB3178034 完成的 win7 系统可能出现 D3Dcompiler_47 丢失，让 WPF 等软件无法启动

<!--more-->
<!-- CreateTime:2019/11/29 8:22:10 -->

<!-- csdn -->
<div id="toc"></div>

## 现象

现象是无法启动，可以在事件查看器看到日志

```csharp
错误应用程序名称: xx.exe，版本: 5.1.3.33526，时间戳: 0x59c5951c
错误模块名称: KERNELBASE.dll，版本: 6.1.7601.17514，时间戳: 0x4ce7bafa
异常代码: 0xe0434352
错误偏移量: 0x0000b727
错误进程 ID: 0x8c
错误应用程序启动时间: 0x01d339ce8c34bedb
错误应用程序路径: xx
错误模块路径: C:\Windows\syswow64\KERNELBASE.dll
报告 ID: cca5651f-a5c1-11e7-9921-00155d356504
```

## 调用堆栈

```
Framework Version: v4.0.30319
Description: The process was terminated due to an unhandled exception.
Exception Info: System.ComponentModel.Win32Exception

Exception Info: System.DllNotFoundException
at MS.Internal.NativeWPFDLLLoader.LoadNativeWPFDLL(UInt16*, UInt16*)
at MS.Internal.NativeWPFDLLLoader.LoadCommonDLLsAndDwrite()
at <Module>.CModuleInitialize.{ctor}(CModuleInitialize*, Void ())
at <Module>.?A0x721f77f1.CreateCModuleInitialize()
```

## 解决方法

安装[在 Windows Server 2012、Windows Server 7 和 Windows Server 2008 R2 上的 d3dcompiler_47.dll 组件的更新](https://support.microsoft.com/zh-cn/help/4019990/update-for-the-d3dcompiler-47-dll-component-on-windows )

注意，此时卸载重装 .NET 4.5 可以解除依赖，但是如果用到像素着色器依然会提示文件损坏

## 复现步骤

step1：安装 .NET 4.6 (4.6、4.6.1、4.6.2都会出现这个问题)

step2：安装以下两个更新：KB4040973 KB3178034 （任意安装顺序）；

说明：

1、KB3178034 是 Microsoft 图形组件安全更新程序；发布时间：2016 年 8 月 9 日

[MS16-097: Description of the security update for Microsoft Graphics Component: August 9, 2016](https://support.microsoft.com/en-us/help/3178034/ms16-097-description-of-the-security-update-for-microsoft-graphics-com )

[Microsoft 安全公告 MS16-097 - 严重](https://docs.microsoft.com/zh-cn/security-updates/Securitybulletins/2016/ms16-097?redirectedfrom=MSDN )

2、KB4040973 是 net46以上 相关更新程序；发布时间：2017 年 9 月 12 日

[Description of the Security and Quality Rollup for the .NET Framework 4.6, 4.6.1, 4.6.2, and 4.7 for Windows 7 SP1 and Windows Server 2008 R2 SP1 and for the .NET Framework 4.6 for Windows Server 2008 SP2: September 12, 2017](https://support.microsoft.com/en-us/help/4040973/description-of-the-security-and-quality-rollup-for-the-net-framework-4 )

3、上述更新安装后，计算机上并不会出现 D3Dcompiler_47.dll ，但引入了其依赖；

4、上述更新必须同时安装，只安装其中一个不会出现问题。

5、出现这个问题之后，重新安装.NET4.6，或者升级 .NET4.6 为 4.6.1或4.6.2不能解决问题。

## 影响范围

- 用 .NET 4.5 和以上版本的 WPF 程序
- 其他用到像素着色器的 win32 程序

## 相关链接

[win7系统电脑丢失D3DCOMPILER_47.DLL 怎么办 - Microsoft Community](https://answers.microsoft.com/zh-hans/windows/forum/all/win7%E7%B3%BB%E7%BB%9F%E7%94%B5%E8%84%91%E4%B8%A2/85dec42a-f0ed-4b16-bb57-fd838ca1a49c?auth=1 )

[启动时出现 Photoshop 系统错误 - 缺少 D3DCOMPILER_47.dll](https://helpx.adobe.com/cn/photoshop/kb/photoshop-error-launch-d3dcompiler.html )

[WPF程序停止工作-CSDN论坛](https://bbs.csdn.net/topics/392423671 )

## 官方措施

在 .NET Core 版本修复

[Adding d3d_compiler dependency to known issues by rladuca · Pull Request #190 · dotnet/wpf](https://github.com/dotnet/wpf/pull/190 )

[WPF Applications require crash with System.TypeLoadException when VC++ redistributables are not present · Issue #37 · dotnet/wpf](https://github.com/dotnet/wpf/issues/37 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
