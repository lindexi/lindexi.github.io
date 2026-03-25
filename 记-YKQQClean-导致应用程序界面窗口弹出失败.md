
# 记 YKQQClean 导致应用程序界面窗口弹出失败

本文记录 YKQQClean 的 Shell 注入导致应用程序界面窗口弹出失败的问题

<!--more-->


<!-- CreateTime:2026/03/25 07:12:14 -->

<!-- 发布 -->
<!-- 博客 -->

这是 成都星汉云科科技有限公司 提供的工具软件，这个软件有添加 Shell 扩展，安装了此程序可能导致某些应用程序界面窗口弹出失败

从 <https://gitee.com/softcnkiller/data/blob/master/folder.txt> 收集列表可以知道，此软件在清单之内，可以放心删掉

其中的 YKQQCleanShell64.dll 有 Shell 注入，需要先用 ShellView 工具在管理员权限下禁用，然后重启 explorer 进程后删除，其他的文件都可以直接结束进程删除




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。