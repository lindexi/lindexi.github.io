---
title: Roslyn 分析器 EnforceExtendedAnalyzerRules 属性的作用
description: 在开始编写 dotnet 的 Roslyn 分析器项目时，会被 VisualStudio 通过 RS1036 要求在项目文件配置上 EnforceExtendedAnalyzerRules 属性，本文将和大家介绍 EnforceExtendedAnalyzerRules 属性的作用
tags: Roslyn,MSBuild,编译器,SourceGenerator,生成代码
category: 
---

<!-- CreateTime:2023/6/7 8:54:59 -->


<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：Roslyn,MSBuild,编译器,SourceGenerator,生成代码 -->

根据 Roslyn 分析器项目要求，需要在 csproj 项目文件添加 EnforceExtendedAnalyzerRules 属性的设置，如以下代码

```xml
    <EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>
```

设置完成之后的 csproj 项目文件代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>

    <EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>

    <LangVersion>latest</LangVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.4" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.6.0" PrivateAssets="all" />
  </ItemGroup>

</Project>
```

现在 RS1036 只是一个警告而已，可以无视

设置 EnforceExtendedAnalyzerRules 为 true 的作用是带入 banned API 分析的功能，则提供代码编写过程中提示不适用于 Roslyn 分析器项目使用的 API 分析功能

设置 EnforceExtendedAnalyzerRules 为 true 时，有以下的 API 将会被提示不可用

```
T:System.IO.File; Do not do file IO in analyzers
T:System.IO.Directory; Do not do file IO in analyzers
M:System.IO.Path.GetTempPath; Do not do file IO in analyzers
T:System.Environment; Analyzers should not read their settings directly from environment variables
M:System.Reflection.Assembly.Load(System.Byte[]); Analyzers should only load their dependencies via standard runtime mechanisms
M:System.Reflection.Assembly.Load(System.String); Analyzers should only load their dependencies via standard runtime mechanisms
M:System.Reflection.Assembly.Load(System.Reflection.AssemblyName); Analyzers should only load their dependencies via standard runtime mechanisms
M:System.Reflection.Assembly.Load(System.Byte[],System.Byte[]); Analyzers should only load their dependencies via standard runtime mechanisms
P:System.Globalization.CultureInfo.CurrentCulture; Analyzers should use the locale given by the compiler command line arguments, not the CurrentCulture
P:System.Globalization.CultureInfo.CurrentUICulture; Analyzers should use the locale given by the compiler command line arguments, not the CurrentUICulture
T:Microsoft.CodeAnalysis.GeneratorInitializationContext; Non-incremental source generators should not be used, implement IIncrementalGenerator instead
T:Microsoft.CodeAnalysis.GeneratorExecutionContext; Non-incremental source generators should not be used, implement IIncrementalGenerator instead
```

详细请看： <https://raw.githubusercontent.com/dotnet/roslyn-analyzers/2b6ab8d727ce73a78bcbf026ac75ea8a7c804daf/src/Microsoft.CodeAnalysis.Analyzers/Core/AnalyzerBannedSymbols.txt>

如上文描述，设置 EnforceExtendedAnalyzerRules 为 true 的作用就是提供 API 禁用分析功能，防止写出分析器不支持的代码

更多关于此的讨论请参阅 [https://github.com/dotnet/roslyn/issues/63290](https://github.com/dotnet/roslyn/issues/63290 )
