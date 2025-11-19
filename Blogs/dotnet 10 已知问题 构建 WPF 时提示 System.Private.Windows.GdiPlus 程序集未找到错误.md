---
title: dotnet 10 已知问题 构建 WPF 时提示 System.Private.Windows.GdiPlus 程序集未找到错误
description: 本文记录 dotnet 10 的已知问题，将会导致 WPF 项目构建的时候给出错误
tags: WPF,dotnet
category: 
---

<!-- CreateTime:2025/11/15 07:29:18 -->

<!-- 发布 -->
<!-- 博客 -->

此问题开始是在 <https://github.com/dotnet/wpf/issues/11246> 被报告的

此问题已修复，详细请看 <https://github.com/dotnet/dotnet/pull/3120>

本文写于 2025年11月15日 此时为 .NET 10 首个正式版本发布，此问题为首个正式版本带来的。此问题在首个正式版本发布之后才修复，需要等待下个版本 SDK 发布才能解决此问题

详细错误信息如下

```
error CS0012: The type 'IImage' is defined in an assembly that is not referenced. You must add a reference to assembly 'System.Private.Windows.GdiPlus, Version=10.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'
```

如果等不及 SDK 更新，可使用以下补丁方法：

用户修复构建：

```xml
  <!-- Workaround for https://github.com/dotnet/wpf/issues/11246 -->
  <!-- Copy from https://github.com/dotnet/sdk/issues/51173#issuecomment-3392609295 -->
  <Target Name="_AddDrawingReference" AfterTargets="ResolveTargetingPackAssets">
    <ItemGroup>
      <Reference Condition="'%(Reference.FileName)' == 'PresentationCore'" 
                 Include="%(Reference.RootDir)%(Reference.Directory)System.Drawing.Common.dll;
                          %(Reference.RootDir)%(Reference.Directory)System.Private.Windows.GdiPlus.dll" />
    </ItemGroup>
  </Target>
```

用于自包含 self contained 应用：

```xml
  <!-- Copy from https://github.com/dotnet/windowsdesktop/issues/5405#issuecomment-3548429691 -->
  <Target Name="_AddDrawingRuntime" 
          AfterTargets="ResolveRuntimePackAssets"
          Condition="'$(SelfContained)' == 'true' AND '$(UseWPF)' == 'true' AND '$(UseWinforms)' != 'true'">
    <ItemGroup>
      <_additionalRuntimePackAsset Condition="'%(RuntimePackAsset.FileName)' == 'PresentationCore'" 
        Include="@(RuntimePackAsset->'%(RootDir)%(Directory)System.Drawing.Common.dll')" DestinationSubPath="System.Drawing.Common.dll" />
      <_additionalRuntimePackAsset Condition="'%(RuntimePackAsset.FileName)' == 'PresentationCore'" 
        Include="@(RuntimePackAsset->'%(RootDir)%(Directory)System.Private.Windows.GdiPlus.dll')" DestinationSubPath="System.Private.Windows.GdiPlus.dll" />
      <RuntimePackAsset Include="@(_additionalRuntimePackAsset)" />
      <ReferenceCopyLocalPaths Include="@(_additionalRuntimePackAsset)" />
    </ItemGroup>
  </Target>
```

等待下个 .NET 10 正式版本发布之后，以上补丁代码需要删掉

参考链接：

- https://github.com/dotnet/wpf/issues/11261
- https://github.com/dotnet/sdk/issues/51173
- https://github.com/NuGet/Home/issues/11786
- https://github.com/dotnet/windowsdesktop/issues/5405
- https://github.com/dotnet/dotnet/pull/3402
