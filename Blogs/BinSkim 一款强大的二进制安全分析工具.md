---
title: BinSkim 一款强大的二进制安全分析工具
description: 在现代软件开发中，确保应用程序的安全性是一个至关重要的任务。为了识别潜在的安全问题，开发者需要使用专业工具对二进制文件进行深入分析。BinSkim 就是一款专为此设计的开源工具，它能够对可移植可执行文件（PE 文件）及其关联的符号文件（PDB）进行检查，从而发现多种安全问题
tags: 
category: 
---

<!-- 发布 -->
<!-- 博客 -->

本文由 GPT-4o 辅助编写

## 什么是 BinSkim？

BinSkim 是由微软开源的一款静态分析工具，主要用于扫描 Windows 的 PE 文件（如 DLL 和 EXE 文件），以识别以下常见的安全问题：

- **过时的编译器工具集**：二进制文件应尽可能使用最新的编译器工具集，以利用当前操作系统和编译器提供的安全缓解措施。
- **不安全的编译设置**：二进制文件应采用最安全的编译设置，以启用操作系统提供的安全功能，并最大化编译器错误和警告报告。
- **签名问题**：已签名的二进制文件应使用强加密算法进行签名。

通过这些检查，BinSkim 能帮助开发者发现潜在的漏洞并提升软件整体安全性。

---

## 如何获取 BinSkim？

BinSkim 是一个开源项目，你可以通过以下途径获取它：

1. 在 [GitHub](https://github.com/Microsoft/binskim) 上查看项目源码。
2. 从 [NuGet](https://www.nuget.org/packages/Microsoft.CodeAnalysis.BinSkim/) 获取最新版本。

下载 NuGet 包之后，用压缩工具将 nupkg 解压缩，即可在 tools\net9.0\win-x64\ 文件夹里找到 BinSkim.exe 文件

---

## 如何运行 BinSkim？

### 快速开始

BinSkim 的主要功能是分析 Windows PE 文件，例如动态链接库（DLL）和可执行文件（EXE）。以下是一些常见使用场景：

1. **分析单个文件**  
   例如，分析位于 `c:\temp` 路径下名为 `MyProjectFile.dll` 的文件：
   ```bash
   binskim.exe analyze c:\temp\MyProjectFile.dll
   ```

2. **递归分析多个文件**  
   分析当前目录及其子目录下所有扩展名为 `.dll` 或 `.exe` 的文件：
   ```bash
   binskim analyze *.exe *.dll --recurse true
   ```

3. **生成 SARIF 格式日志**  
   分析当前目录下所有 `.dll` 文件，并将结果输出到指定的 SARIF 日志文件中：
   ```bash
   binskim analyze *.dll --output MyLog.sarif
   ```

---

### 常用命令参考

| 命令类型 | 描述 |
| -------- | ---- |
| **通用帮助** | 显示 BinSkim 支持的所有内置命令，例如 `help`、`analyze` 和 `capture` 等。运行命令：`binskim.exe --help` |
| **详细帮助** | 查看特定命令的详细说明。例如：<ul><li>`binskim.exe help analyze`</li><li>`binskim.exe help exportRules`</li></ul> |

---

### Analyze 命令参数详解

在运行 `analyze` 命令时，可以使用以下附加参数来配置扫描行为：

| 参数 (缩写/全称) | 含义 |
| ---------------- | ---- |
| **--trace** | 指定执行跟踪信息，用分号分隔。例如：`"PdbLoad;ScanTime"`。 |
| **--sympath** | 提供符号路径，用于定位 PDB 文件。例如：`SRV*https://msdl.microsoft.com/download/symbols`。|
| **--local-symbol-directories** | 指定本地目录路径，用于查找 PDB 文件。|
| **-o, --output** | 指定 SARIF 格式日志输出路径。|
| **-r, --recurse [true\|false]** | 是否递归分析子目录中的文件。|
| **-c, --config** | 指定配置文件路径，用于自定义扫描规则（默认值为 `default`）。|
| **-q, --quiet [true\|false]** | 如果为 true，则不在控制台打印结果，仅输出到日志文件中。|
| **-s, --statistics** | 生成统计信息，例如执行时间和有效目标数量等。|
| **--insert** | 将额外数据插入到日志中，例如环境变量、哈希值等。|
| **-e, --environment [true\|false]** | 是否记录运行环境信息（可能包含敏感数据）。|
| **--rich-return-code [true\|false]** | 输出详细返回码，以标识执行状态。|

---

### 示例参数解析

#### `--sympath`
该参数用于指定符号服务器路径或本地符号缓存位置。例如：
```bash
binskim analyze myfile.dll --sympath "Cache*d:\symbols;Srv*https://symweb"
```
> 注意：如果未正确加载 PDB 文件，BinSkim 会生成错误代码 `ERR97`，提示具体原因。

#### `--local-symbol-directories`
如果你的构建系统将 PDB 文件存放在其他位置，可以通过该参数指定本地目录。例如：
```bash
binskim analyze myfile.dll --local-symbol-directories "d:\pdbs"
```

#### `-o, --output`
将扫描结果保存为 SARIF 格式日志，可用于进一步处理或集成到 IDE 中，如 Visual Studio。

#### 返回码说明
BinSkim 提供了详细的返回码，可帮助开发者快速定位问题。例如：
- `0x80000000`: 执行成功但发现一个或多个错误。
- `0x40000000`: 执行成功但发现警告。
- `0x20`: 无法加载 PDB 文件。

完整返回码列表请参考官方文档。

---

## BinSkim 支持哪些规则？

BinSkim 提供了一系列内置规则，用于检查不同方面的问题。例如：

| 规则 ID | 名称 | 检查内容 |
| ------- | ---- | -------- |
| BA2002  | 避免使用易受攻击的依赖项 | 检查所有链接模块源代码 |
| BA2006  | 使用安全工具构建       | 检查编译器版本           |
| BA2011  | 启用堆栈保护           | 检查是否启用了 `/GS` 标志 |

完整规则列表和详细说明可在 [官方文档](https://github.com/microsoft/binskim/blob/master/docs/RulesAndErrorsTroubleshootingGuide.md) 中查看。

---

## 总结

BinSkim 是一款功能强大的二进制分析工具，通过静态扫描帮助开发者发现潜在的安全问题。从过时的编译器工具集到不安全的编译设置，再到签名问题，BinSkim 覆盖了多种关键场景。同时，其灵活的参数配置和详细的返回码设计，使得它能够轻松集成到自动化构建流水线中，为软件提供更高层次的安全保障。

如果你正在寻找一款高效且专业的二进制静态分析工具，不妨试试 BinSkim！立即前往 [GitHub](https://github.com/Microsoft/binskim) 或 [NuGet](https://www.nuget.org/packages/Microsoft.CodeAnalysis.BinSkim/) 下载并体验吧！
