# 记 YKQQClean 导致应用程序界面窗口弹出失败

本文记录 YKQQClean 的 Shell 注入导致应用程序界面窗口弹出失败的问题

<!--more-->
<!-- CreateTime:2026/03/25 07:12:14 -->

<!-- 发布 -->
<!-- 博客 -->

这是 成都星汉云科科技有限公司 提供的工具软件，这个软件有添加 Shell 扩展，安装了此程序可能导致某些应用程序界面窗口弹出失败

从 <https://gitee.com/softcnkiller/data/blob/master/folder.txt> 收集列表可以知道，此软件在清单之内，可以放心删掉

其中的 YKQQCleanShell64.dll 有 Shell 注入，需要先用 ShellView 工具在管理员权限下禁用，然后重启 explorer 进程后删除，其他的文件都可以直接结束进程删除