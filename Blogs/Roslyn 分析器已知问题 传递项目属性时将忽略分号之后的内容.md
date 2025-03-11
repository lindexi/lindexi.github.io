---
title: Roslyn 分析器已知问题 传递项目属性时将忽略分号之后的内容
description: 本文记录 Roslyn 分析器、源代码生成器的已知问题，通过CompilerVisibleProperty 传递值时，所有在 `;`、`#` 和换行符之后的字符都会被忽略
tags: Roslyn
category: 
---

<!-- 发布 -->
<!-- 博客 -->

相关问题链接：

- <https://github.com/dotnet/roslyn/issues/43970>
- <https://github.com/dotnet/roslyn/issues/51692>

此问题由 [walterlv](https://github.com/walterlv) 发现，我只是一个记录问题的工具人

## 解决方案

将需要传递的值进行 Base64 编码，避免 `;`、`#` 和换行符的影响。

此方案来自于： <https://github.com/dotnet/roslyn/issues/51692>

不足之处，这是一个十分奇怪的行为，先 Base64 编码，再 Base64 解码。这个过程只是为了断开表达式关系，代码也不优雅

### 其他无效的解决方案

1.是否可以通过 `Replace` 来替换掉 `;` 字符？

经过测试，此方案仅在原属性是字符串常量时才正常工作；当原属性是 MSBuild 计算出来的值时，`Replace` 会导致把计算过程表达式当作字符串常量来处理，导致替换完之后表达式变成了字符串结果了。

2.是否可在 Target 中替换字符？

经过测试，在多个 Target 中替换都会得到跟上述相同的结果。

3.是否可通过 `$([MSBuild]::Escape('$(Foo)'))` 转义来规避？

经过测试，会得到跟上述相同的结果

## 文档

微软官方的 CookBook 中完全没有提及此问题：

- <https://github.com/dotnet/roslyn/blob/main/docs/features/incremental-generators.cookbook.md>

有网友主动为 CookBook 提交了此问题的规避方案： <https://github.com/dotnet/roslyn/pull/76818>

不过此规避方案也仅在字符串常量时有效，不适用于本方案
