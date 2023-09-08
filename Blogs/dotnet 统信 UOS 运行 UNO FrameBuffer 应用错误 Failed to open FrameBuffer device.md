本文记录在 UOS 统信系统上运行 UNO 的基于 Skia 的 FrameBuffer 应用报错问题，错误提示是 Unhandled exception. System.InvalidOperationException: Failed to open FrameBuffer device /dev/fb0 (13) 的问题。问题原因是 UNO 应用的 FrameBuffer 写入失败，本文将告诉大家调查方法

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

首先需要先确定所运行的 UOS 系统是否正常，请根据 [Linux Framebuffer 实验 - 浇筑菜鸟 - 博客园](https://www.cnblogs.com/jzcn/p/16898249.html ) 博客的方法进行测试

如果执行 `dd if=/dev/zero of=/dev/fb0` 命令提示 dd: 打开 '/dev/fb0' 失败，权限不够，则可能只是权限问题，先使用 `sudo su` 命令提权，再次执行命令

如果命令可以执行成功，证明只是因为权限问题而无法运行 UNO 应用程序而已

尝试在以上已提权的前提下，再次控制台执行 UNO 应用程序，看是否能够执行成功。理论上是能够成功的，推荐此时测试使用简单的 demo 程序，比如我编写的使用 UNO 官方默认应用程序，代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/9285ee59071c54b49dd6ad0e868a744b4998d203/FayjarbeelajoFalfarkeyi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/9285ee59071c54b49dd6ad0e868a744b4998d203/FayjarbeelajoFalfarkeyi) 上，可以通过以下方式获取整个项目的代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 9285ee59071c54b49dd6ad0e868a744b4998d203
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 9285ee59071c54b49dd6ad0e868a744b4998d203
```

获取代码之后，进入 FayjarbeelajoFalfarkeyi 文件夹

如果是本身系统问题，还请尝试解决系统问题哈，这部分属于通用的 UOS 问题。我对 Linux 了解也很少，就不在这里乱说了。详细请看 [Linux图形界面基础知识](https://huangwang.github.io/2018/12/09/Linux%E5%9B%BE%E5%BD%A2%E7%95%8C%E9%9D%A2%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/ ) 和 [信创终端之Linux桌面系统：原生桌面 vs 定制魔改_沅陵县信创工作计算机终端更换-CSDN博客](https://blog.csdn.net/McwoLF/article/details/107139290 )
