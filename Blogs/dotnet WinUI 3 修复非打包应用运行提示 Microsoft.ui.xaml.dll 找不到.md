---
title: dotnet WinUI 3 修复非打包应用运行提示 Microsoft.ui.xaml.dll 找不到
description: 本文记录一个 WinUI 3 的坑点，有时候开发者只是想拉下来代码跑一下，自己本机没有预先安装好 Windows App Runtime 导致缺失环境，进而在运行的时候提示 Unable to load DLL 找不到 Microsoft.ui.xaml.dll 启动失败

<!--more-->

tags: dotnet
category: 
---

<!-- CreateTime:2024/08/27 07:11:30 -->

<!-- 发布 -->
<!-- 博客 -->

详细的错误信息如下

System.DllNotFoundException:“Unable to load DLL 'Microsoft.ui.xaml.dll' or one of its dependencies: 找不到指定的模块。 (0x8007007E)”

大部分情况下的错误都在如以下代码的 XamlCheckProcessRequirements 里面

```csharp
        [global::System.Runtime.InteropServices.DllImport("Microsoft.ui.xaml.dll")]
        [global::System.Runtime.InteropServices.DefaultDllImportSearchPaths(global::System.Runtime.InteropServices.DllImportSearchPath.SafeDirectories)]
        private static extern void XamlCheckProcessRequirements();
```

对应的英文错误信息如下

Unable to load DLL 'Microsoft.ui.xaml.dll' or one of its dependencies: The specified module could not be found. (0x8007007E)

此问题是因为没有安装好 Windows App Runtime 环境，导致 WinUI 3 缺失环境而运行失败，或者是因为没有将 WinUI 3 使用自包含方式发布而找不到环境

对应以上原因就有两个修复方法：

方法一是给安装上环境，安装方法如下

进入到 [https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/downloads](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/downloads) 下载页

下载安装 Windows App Runtime 安装包 WindowsAppRuntimeInstall.exe 文件修复此问题

方法二是构建为自包含方式，在 csproj 里面添加如下代码即可

```xml
   <WindowsAppSDKSelfContained>true</WindowsAppSDKSelfContained>
```

详细请参考 [WinUI 3 修复非打包应用运行提示缺少 Windows App Runtime 环境](https://blog.lindexi.com/post/WinUI-3-%E4%BF%AE%E5%A4%8D%E9%9D%9E%E6%89%93%E5%8C%85%E5%BA%94%E7%94%A8%E8%BF%90%E8%A1%8C%E6%8F%90%E7%A4%BA%E7%BC%BA%E5%B0%91-Windows-App-Runtime-%E7%8E%AF%E5%A2%83.html )

此问题已经报告给官方，详细请看 <https://github.com/microsoft/WindowsAppSDK/issues/2443>
