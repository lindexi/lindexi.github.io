---
title: WPF 设置 IncludePackageReferencesDuringMarkupCompilation 属性导致分析器不工作
description: 本文记录在 WPF 项目里面设置 IncludePackageReferencesDuringMarkupCompilation 属性为 False 导致了项目所安装的分析器不能符合预期工作

<!--more-->

tags: WPF
category: 
---

<!-- CreateTime:2023/8/2 19:37:40 -->

<!-- 发布 -->
<!-- 博客 -->

设置 IncludePackageReferencesDuringMarkupCompilation 属性为 false 将配置 WPF 在构建 XAML 过程中创建的 tmp.csproj 过程中将不引用依赖的 nuget 包。分析器默认也是通过 nuget 包方式安装的，这就导致了分析器项目没有被 tmp.csproj 项目正确使用到

如果项目里面有代码依赖分析器生成的影响语义的代码，那这部分代码将会构建不通过

详细关于 IncludePackageReferencesDuringMarkupCompilation 属性请参阅 [WPF 修复 dotnet 6 与源代码包冲突](https://blog.lindexi.com/post/WPF-%E4%BF%AE%E5%A4%8D-dotnet-6-%E4%B8%8E%E6%BA%90%E4%BB%A3%E7%A0%81%E5%8C%85%E5%86%B2%E7%AA%81.html )
