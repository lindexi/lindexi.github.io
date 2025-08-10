---
title: VisualStudio 2022 找不到内存 反汇编 寄存器调试工具
description: 本文将告诉大家如何解决在 VisualStudio 2022 的 调试-窗口 里面找不到内存、 反汇编、 寄存器这三个调试工具的问题
tags: VisualStudio
category: 
---

<!-- CreateTime:2024/1/11 20:14:56 -->

<!-- 发布 -->
<!-- 博客 -->

找不到的原因是没有启用地址级调试

只需要在“工具”（或“调试”）>“选项”>“调试”中选择“启用地址级调试” 然后进行调试即可看到

<!-- ![](image/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具1.png) -->
![](https://img2024.cnblogs.com/blog/1080237/202508/1080237-20250810094833492-1263364848.png)

开启之后，即可在 调试-窗口 里面找到内存、 反汇编、 寄存器这三个调试工具

<!-- ![](image/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具0.png) -->

![](https://img2024.cnblogs.com/blog/1080237/202508/1080237-20250810094833812-1578505225.png)

参考文档：[在调试器中查看寄存器值 - Visual Studio (Windows) Microsoft Learn](https://learn.microsoft.com/zh-cn/visualstudio/debugger/how-to-use-the-registers-window?view=vs-2022 )
