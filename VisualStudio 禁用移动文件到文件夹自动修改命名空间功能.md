# VisualStudio 禁用移动文件到文件夹自动修改命名空间功能

在 VisualStudio 2022 里的某个版本开始，将会在移动文件到其他文件夹时，自动修改命名空间，使用匹配文件夹路径的命名空间。如果这个功能能顺手将其他引用此类型的全部符号同时变更，那自然是很好的功能，可惜没有，很多时候都只是修改了移动的文件里面的命名空间，没有更改其他相关引用的代码的逻辑，导致了移动一次文件需要重新将命名空间改回来修复构建，极大降低效率

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

关闭此智能(zhang)的功能的方法是：进入 工具->选项 然后进入项目和解决方案选项卡里面的常规选项卡，去掉移动文件时启用命名空间更新选项即可。对应英文版本的是在 `Projects and Solutions`->`General` 的 `Enable namespace update when moving files` 选项

中文版：

<!-- ![](image/VisualStudio 禁用移动文件到文件夹自动修改命名空间功能/VisualStudio 禁用移动文件到文件夹自动修改命名空间功能1.png) -->

![](http://image.acmx.xyz/lindexi%2FQQ%25E5%259B%25BE%25E7%2589%258720220831103818.png)

英文版：

<!-- ![](image/VisualStudio 禁用移动文件到文件夹自动修改命名空间功能/VisualStudio 禁用移动文件到文件夹自动修改命名空间功能0.png) -->

![](http://image.acmx.xyz/lindexi%2F20228311035501070.jpg)

