---
title: dotnet 9 WPF 连字符渲染支持
description: 连字符渲染功能是 WPF 呼声很高的一个功能，核心需求方是使用 Visual Studio 的开发者们。开发者们期望使用连字符的时候可以进行连字符渲染，让开发过程中的视觉效果更加。于是作为 Visual Studio 的基础设施的 WPF 就要承担此功能的实现。经过漫长的开发，终于在 dotnet 9 里面加入了此功能
tags: WPF,dotnet
category: 
---

<!-- 发布 -->
<!-- 博客 -->

如下图是 dotnet 9 之前的渲染效果界面图：

<!-- ![](image/dotnet 9 WPF 连字符渲染支持/dotnet 9 WPF 连字符渲染支持0.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202411/1080237-20241114073112129-544494365.png)

如下图是 dotnet 9 版本的渲染效果界面图：

<!-- ![](image/dotnet 9 WPF 连字符渲染支持/dotnet 9 WPF 连字符渲染支持1.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202411/1080237-20241114073112591-363222686.png)

## 如何启用或关闭此功能

默认升级到 dotnet 9 即可自动开启

如在 dotnet 9 版本上，期望能够关闭此功能，还原到旧版本的渲染行为，可通过 `Switch.System.Windows.DisableSpecialCharacterLigature` 进行设置

```csharp
        public App()
        {
            AppContext.SetSwitch("Switch.System.Windows.DisableSpecialCharacterLigature", true);
        }
```

## 此功能是如何实现的

此功能是在不开源的 PresentationNative.dll 里面实现的，因此没有直接的实现代码。但预计只是调用 DirectX 的科技即可实现。从哪可以了解到其实现信息？请参阅： [Adding AppContext Switch for disabling special character ligatures by Kuldeep-MS · Pull Request #8990 · dotnet/wpf](https://github.com/dotnet/wpf/pull/8990 )

更多实现 dotnet 9 的 WPF 更新请参阅：[What's new in WPF for .NET 9 - WPF .NET](https://learn.microsoft.com/zh-cn/dotnet/desktop/wpf/whats-new/net90?view=netdesktop-9.0 )
