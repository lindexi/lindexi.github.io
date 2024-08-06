本文将告诉大家如何解决在 VisualStudio 2022 的 调试-窗口 里面找不到内存、 反汇编、 寄存器这三个调试工具的问题

<!--more-->


<!-- CreateTime:2024/1/11 20:14:56 -->

<!-- 发布 -->
<!-- 博客 -->

找不到的原因是没有启用地址级调试

只需要在“工具”（或“调试”）>“选项”>“调试”中选择“启用地址级调试” 然后进行调试即可看到

<!-- ![](image/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具1.png) -->
![](http://cdn.lindexi.site/lindexi%2F2024111201747659.jpg)

开启之后，即可在 调试-窗口 里面找到内存、 反汇编、 寄存器这三个调试工具

<!-- ![](image/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具/VisualStudio 2022 找不到内存 反汇编 寄存器调试工具0.png) -->

![](http://cdn.lindexi.site/lindexi%2F20241112017237286.jpg)

参考文档：[在调试器中查看寄存器值 - Visual Studio (Windows) Microsoft Learn](https://learn.microsoft.com/zh-cn/visualstudio/debugger/how-to-use-the-registers-window?view=vs-2022 )
