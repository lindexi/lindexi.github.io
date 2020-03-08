# 为何使用 DirectComposition

本文主要翻译[Why use DirectComposition](https://msdn.microsoft.com/en-us/library/windows/desktop/hh449195(v=vs.85).aspx )，介绍 DirectComposition 的功能和优点。

<!--more-->
<!-- CreateTime:2019/3/8 8:56:09 -->

<!-- csdn -->

<div id="toc"></div>

<!-- 标签：windows，DirectComposition -->

## 创建更加迷人的界面

虽然界面创建好看是设计师的能力，但是如何可以提高性能，支持更多的动画，这时设计师才可以做出更好的界面。

基于 windows 的软件可以通过 DirectComposition 组合 Visual 和对 Visual 做动画来创建迷人的界面，通过这个技术可以创建独一无二的视觉体验。

虽然看起来很多界面框架都这样说，但是 DirectComposition 从字面看就是 组合的Direct。

## 流畅丰富的动画

现在的界面如果没有动画，一般是很难做出好的体验。如果有很好的动画，但是性能很差，用户也会觉得程序员可以祭天。如何使用 DirectComposition ，可以获得高性能的位图组合引擎，因为有硬件加速。对于帧率要求高的动画也可以使用 DirectComposition ，可以支持高速绘制、缩放和组合很多动画。

如果使用 DirectComposition ，那么 UI 线程不会因为渲染而 starve ，因为 DirectComposition 是在一个专用的线程运行和程序的 UI 线程分离。所以在做复杂动画不需要担心主线程无法处理。

## 组合不同的位图

很多的 Windows 程序都组合几个渲染技术，如界面菜单使用 GDI 来画，因为画静态 GDI 性能好。画动画使用 D3D 来画，然后把多个渲染进行组合，显示在一个窗口，这时就可以使用 DirectComposition 来组合位图。

如果使用 DirectComposition 就不需要关注不同渲染框架内容重叠时的处理，可以把不同位图渲染在相同的层级或子窗口。

## 通过集成 DWM 节省内存

实际 DirectComposition 通过集成 DWM 来创建组合位图和动画显示到屏幕，所以使用 DirectComposition 不需要再安装其他的渲染框架。

需要知道的是 DirectComposition 不是基于 DWM 而是集成，而且 DirectComposition 没有渲染元素的能力而是对渲染完成的位图进行组合。

那么 DWM 是什么作用，DWM 实际作用 Windows 组合引擎或合成程序，需要每个窗口把显示的内容给屏外表面或缓冲区，缓冲区是系统给每个顶层窗口分配的，所有的 GDI、D3D、D2D 到先渲染到这里。然后 DWM 决定如何显示，是组合窗口还是做特效，最后再把缓存放到显卡。

## 兼容原有代码

如果使用 DirectComposition 需要修改之前的界面代码，那么是一个很大的工作，好在使用 DirectComposition 可以支持以前的界面代码。而且 DirectComposition 可以对之前写的界面代码进行组合和动画，从而做出好看的界面。所以可以通过 DirectComposition 对界面进行美化不需要修改原有的代码。

参见：[Why use DirectComposition](https://msdn.microsoft.com/en-us/library/windows/desktop/hh449195(v=vs.85).aspx )

注意，在 Windows 8 和以上的系统才能使用 DirectComposition 的 API 包括在 x86 x64 和 ARM 等系统

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
