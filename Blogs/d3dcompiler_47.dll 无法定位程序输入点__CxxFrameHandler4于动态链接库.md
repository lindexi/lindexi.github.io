---
title: d3dcompiler_47.dll 无法定位程序输入点__CxxFrameHandler4于动态链接库
description: 本文记录 d3dcompiler_47.dll 挖的一个坑
tags: 
category: 
---

<!-- 发布 -->
<!-- 博客 -->

这里故事需要从 2019 的 3 月开始讲，那时微软在 Visual Studio 2019 Preview 3 引入了一个新的功能，用来减少 x64 上的 C++ 异常处理的二进制大小。这个新的功能就被称为 FH4 也就是本文的主角—— `__CxxFrameHandler4` 方法。细节介绍请参阅 <https://devblogs.microsoft.com/cppblog/making-cpp-exception-handling-smaller-x64/>

引入了 `__CxxFrameHandler4` 有什么问题呢？微软将这个方法加入到了 ucrtbase.dll 文件里面。然而按照 ucrtbase.dll 的初始定位来说，这是一个 Universal C Runtime 通用 C 运行时库，讲道理来说应该保持 API 稳定性，避免版本问题。这个库的引入也是指在解决碎片化、无官方化的 C/C++ 引用问题。这个库是从 2015 年 Win10 刚刚出来的时候就引入的，当时开发者们也很开森，因为其地位就和 C# dotnet 的 BCL 差不多，终于有了通用的定义了。更细节的 UCRT 故事请参阅 <https://devblogs.microsoft.com/cppblog/introducing-the-universal-crt/>

引入 `__CxxFrameHandler4` 到 ucrtbase.dll 文件里面也就挖了大坑了，这就意味着只有某个高版本之后的 ucrtbase.dll 文件才带有 `__CxxFrameHandler4` 方法，这就是一个 API 变更。意味着面向通用 SDK 编程的开发者，如果显式依赖了系统带的 ucrtbase.dll 文件时，将在旧版本的 Windows 系统（1909之前）上，出现找不到 `__CxxFrameHandler4` 的问题

系统带的 ucrtbase.dll 是放在 C:\Windows\System32\ucrtbase.dll 路径下的，这就意味着系统层的 ucrtbase.dll 文件必然存在版本碎片化问题。这个问题不是说安装某个 VC++ 库就能解决的，无论是安装 Visual C++ Redistributable 几的版本，都不能解决这个问题。现在只有系统更新能够更新 ucrtbase.dll 文件。但更可怕的问题是，可能有某些开发者为了修复自己应用程序的问题，就在自己程序安装包安装过程中，将自己的 ucrtbase.dll 拷贝进入到 C:\Windows\System32 文件夹，给系统投毒

在有这些问题的前提下，当 Windows Kits 更新到 10.0.26100 或更高版本的 SDK 时，此 SDK 所带的 `C:\Program Files (x86)\Windows Kits\10\Redist\D3D\x64\d3dcompiler_47.dll` 文件就是动态依赖了 `C:\Windows\System32\ucrtbase.dll` 文件，如以下 [Dependencies](https://github.com/lucasg/Dependencies) 工具截图所见内容

<!-- ![](image/d3dcompiler_47.dll 无法定位程序输入点__CxxFrameHandler4于动态链接库/d3dcompiler_47.dll 无法定位程序输入点__CxxFrameHandler4于动态链接库0.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202504/1080237-20250425085305835-1831999183.png)

其版本信息如下

<!-- ![](image/d3dcompiler_47.dll 无法定位程序输入点__CxxFrameHandler4于动态链接库/d3dcompiler_47.dll 无法定位程序输入点__CxxFrameHandler4于动态链接库2.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202504/1080237-20250425085306888-1350635545.png)

而在此版本之前的 10.0.22621 或更低版本里面 d3dcompiler_47.dll 还是将 ucrtbase 静态打进去的，通过 [Dependencies](https://github.com/lucasg/Dependencies) 工具所见内容如下，可以看到依赖非常少

<!-- ![](image/d3dcompiler_47.dll 无法定位程序输入点__CxxFrameHandler4于动态链接库/d3dcompiler_47.dll 无法定位程序输入点__CxxFrameHandler4于动态链接库1.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202504/1080237-20250425085307217-1324310557.png)

由于 10.0.26100 版本的 d3dcompiler_47.dll 显式要求引用 `C:\Windows\System32\ucrtbase.dll` 文件，这就导致了即使在应用程序同路径下，放入 ucrtbase.dll 文件也不能解决问题

好在 d3dcompiler_47.dll 的 API 兼容性做的很好，现在对于开发者来说，解决方法就是先换用旧的 d3dcompiler_47.dll 版本到自己的项目里面或最终输出路径里面覆盖新的版本。经过考古，发现谷歌的 chromium 也遇到相同的问题，也被相同的问题困扰，以下是 <https://chromium.googlesource.com/chromium/src/+/main/docs/windows_build_instructions.md> 的原文

> WARNING: On sufficiently old versions of Windows (1909 or earlier), dawn or related components may fail with a D3d-related error when using the 26100 SDK. This is because the d3dcompiler_47.dll file in the new SDK attempts to dynamically link versions of the Universal C Runtime which are not present by default on older systems. If you experience these errors, you can either update the UCRT on your system, or install the 22621 SDK and use the d3dcompiler_47.dll file included there, which statically links the UCRT.
>
> 警告：在足够旧的 Windows 版本（1909 或更早版本）上，使用 26100 SDK 时，dawn 或相关组件可能会因 D3d 相关错误而失败。这是因为新 SDK 中的 d3dcompiler_47.dll 文件尝试动态链接在较旧系统上默认不存在的通用 C 运行时版本。如果您遇到这些错误，您可以更新系统上的 UCRT，或者安装 22621 SDK 并使用其中包含的 d3dcompiler_47.dll 文件，该文件会静态链接 UCRT。

我去翻看了我电脑的 Chrome 安装路径下的 d3dcompiler_47.dll 文件，发现用的是 10.0.22621.2428 版本的。如上文所述，此版本的 d3dcompiler_47.dll 文件是静态引用 ucrtbase.dll 的，换句话说就是将 ucrtbase 打到 d3dcompiler_47.dll 里面，不存在引用依赖问题。这就使得 Chrome 依然可以在旧版本系统上运行

特别感谢 [lsj](https://blog.sdlsj.net/ ) 说明，在谷歌打包工具链里面，会去下载旧的 10.0.22621.2428 版本，详细请参阅 <https://chromium.googlesource.com/chromium/src/build/+/7ff4b0206b94e56c015473ca832a8001c2e7f080>

对于普通用户咋办呢？那只有去更新系统的 ucrtbase.dll 文件了，更新方法就是打开系统更新去进行系统更新啦

题外话，似乎微软也意识到了 ucrt 已经被投毒了，原本也为了处理 MSVCRT (Microsoft Visual C++ Runtime)的各种兼容性的 UCRT 也似乎步入后尘。这里的故事有些复杂，细节请参阅 <https://blog.csdn.net/cjmqas/article/details/79296865>

相关链接：

- https://issues.chromium.org/issues/372059340
- https://learn.microsoft.com/zh-cn/cpp/porting/upgrade-your-code-to-the-universal-crt?view=msvc-170
- https://github.com/TGSAN/CMWTAT_Digital_Edition/issues/45
- https://github.com/dotnet/runtime/discussions/96911
- https://stackoverflow.com/questions/65672356/what-is-cxxframehandler4-and-what-does-linker-error-unresolved-external-symbo
- https://github.com/chromium/chromium/blob/main/docs/windows_build_instructions.md

分割线，分割线，分割线

如果你不是遇到 d3dcompiler_47.dll 文件无法定位程序输入点__CxxFrameHandler4于动态链接库而进来的开发者，只是自己的应用程序遇到了 `__CxxFrameHandler4` 找不到或 LNK2022:无法解析的外部符号__CxxFrameHandler4问题而进入本文的，那可以试试代码构建时添加参数进行关闭，添加 ` -d2FH4-` 构建参数，详细请参阅 <https://devblogs.microsoft.com/cppblog/making-cpp-exception-handling-smaller-x64/?commentid=543#comment-543>
