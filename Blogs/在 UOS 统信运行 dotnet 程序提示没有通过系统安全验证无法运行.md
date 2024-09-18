---
title: 在 UOS 统信运行 dotnet 程序提示没有通过系统安全验证无法运行
description: 本文记录 dotnet 应用程序在 UOS 统信系统上运行时，提示 没有通过系统安全验证，无法运行 的问题

<!--more-->

tags: dotnet
category: 
---

<!-- CreateTime:2023/9/7 9:29:42 -->

<!-- 发布 -->
<!-- 博客 -->

这个问题是因为没有开启 UOS 统信的开发者模式，直接将自己构建完成的包放上去跑导致的问题

解决方法十分简单，只需要开启开发者模式即可

控制中心 -> 通用 -> 开发者模式 -> 进入开发者模式 登录，重启

相关问题： [在 UOS 统信安装 dotnet sdk 失败 提示 failed the verification](https://blog.lindexi.com/post/%E5%9C%A8-UOS-%E7%BB%9F%E4%BF%A1%E5%AE%89%E8%A3%85-dotnet-sdk-%E5%A4%B1%E8%B4%A5-%E6%8F%90%E7%A4%BA-failed-the-verification.html )
