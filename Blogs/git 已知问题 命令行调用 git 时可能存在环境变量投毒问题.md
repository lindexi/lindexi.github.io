---
title: git 已知问题 命令行调用 git 时可能存在环境变量投毒问题
description: 本文记录一个我在 git 钩子唤起一个 C# dotnet 的进程，在此进程里面使用 Process.Start 执行 git 命令的时候，被 git 钩子环境变量投毒的问题
tags: git
category: 
---

<!-- CreateTime:2025/03/15 07:15:26 -->

<!-- 发布 -->
<!-- 博客 -->

核心代码非常简单，我只是使用 git add 命令而已

```csharp
    var sourceFolder = @"C:\lindexi\Work\Source\";

            var processStartInfo = new ProcessStartInfo("git")
            {
                ArgumentList =
                {
                    "add",
                    "."
                },
                WorkingDirectory = sourceFolder,
            };

            Process.Start(processStartInfo)!.WaitForExit();
```

在 git 钩子里面调起进程的时候，会额外注入许多环境变量，比如 GIT_INDEX_FILE 和 GIT_DIR 等，这些环境变量是指定的是当前的 git 钩子所在的 git 仓库的

这就导致了我设置了 `WorkingDirectory = sourceFolder` 无效，直接导致 git add 内容到当前 git 钩子所在的仓库，而不是 `sourceFolder` 仓库

解决方法是清空环境变量，解决被 git 钩子投毒

```csharp
            var processStartInfo = new ProcessStartInfo("git")
            {
                ArgumentList =
                {
                    "add",
                    "."
                },
                WorkingDirectory = sourceFolder,
            };
            // 这是在 git 里面调用的，会被注入 git 的环境变量，从而被投毒，如 GIT_INDEX_FILE GIT_DIR 等，导致加入的文件不是在要求的路径
            processStartInfo.Environment.Clear();

            Process.Start(processStartInfo)!.WaitForExit();
```

以上代码核心是使用 `processStartInfo.Environment.Clear();` 清理所有传入到启动进程的环境变量

参考文档：

[Git - Git 钩子](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90 )
