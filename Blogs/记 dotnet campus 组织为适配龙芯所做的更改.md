---
title: 记 dotnet campus 组织为适配龙芯所做的更改
description: 本文记录在龙芯适配过程中的 dotnet campus 开源组织的更改，这些更改仅仅只是 dotnet campus 开源组织自身的项目的更改，不涉及给任何龙芯上游的贡献
tags: dotnet
category: 
---

<!-- 发布 -->
<!-- 博客 -->

## 打包构建工具

- https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/pull/154

修复 deb 包里面记录的架构没有包含龙芯

- https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/pull/155

升级更基础的库命令行调用 git 支持 Linux 龙芯版本

## 分析器版本系列

当前 2024.11 龙芯的最新 dotnet sdk 为 8.0.7 版本，这个版本分析器对应的是 4.8.0 版本。为了支持龙芯构建，降低分析器版本。详细请参阅 https://ftp.loongnix.cn/dotnet/

- https://github.com/dotnet-campus/dotnetCampus.LatestCSharpFeatures/pull/5
- https://github.com/dotnet-campus/dotnetCampus.SourceLocalizations/pull/8
- https://github.com/dotnet-campus/dotnetCampus.Logger/pull/29
- https://github.com/dotnet-campus/dotnetCampus.Ipc/pull/156
