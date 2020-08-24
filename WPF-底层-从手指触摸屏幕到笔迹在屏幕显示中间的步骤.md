
# WPF 底层 从手指触摸屏幕到笔迹在屏幕显示中间的步骤

整个 WPF 就是一个UI框架，一个 UI 框架最重要的是 交互 和 显示 部分，而书写这个功能将会完全贯穿 WPF 整个框架的功能。本文非入门级博客，本文包含了大量链接博客，阅读本文你将会了解从用户手指触摸屏幕到最终屏幕打印出笔迹的应用程序执行的步骤

<!--more-->


<!-- 发布 -->

本文实际内容不多，但是如果加上链接的博客，那么总内容将会非常多，还请小伙伴仔细阅读本文链接的博客

从软件的角度上，可以将触摸屏看成是一个软件制作的驱动组件，因此就可以规避复杂的硬件带来的问题。当前的触摸是有 HID 标准的，任何走标准 HID 设备的硬件设备，只要实现得对，咱上层软件是不需要关注硬件的细节的。更多有关协议部分请看 [Windows 的 Pen 协议](https://blog.lindexi.com/post/Windows-%E7%9A%84-Pen-%E5%8D%8F%E8%AE%AE.html)

规避了硬件设备，此时咱就不需要画精力去了解硬件设备的收集触摸点的机制，以及封装数据和系统的解包是如何做的

在 WPF 的触摸在系统最底层使用的是 RealTime Stylus 机制实现，这个机制能达到比 WM_Touch 触摸消息快非常多倍的接收速度，基本可以认为硬件设备发送到系统瞬间就到应用程序上，中间过程仅有发生几次锁和读取内存数据的时间。从 RealTime Stylus 到 WPF 框架经过 PenIMC 模块，请看 [WPF 触摸底层 PenImc 是如何工作的](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%BA%95%E5%B1%82-PenImc-%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84.html)

而 WPF 尽管可以在 Stylus Input 线程使用 PenThreadWorker 通过 RealTime Stylus 机制快速获取触摸点，但是 WPF 为了让业务逻辑更好实现，此时将会在在主线程触发 Touch 或 Stylus 事件。因此如果监听 Touch 等这些事件，那么将需要等待线程切换和等待主线程忙碌。因此高性能的笔迹实现推荐通过 StylusPlugIn 的方法，在触摸线程获取触摸点，详细请看 [WPF 高速书写 StylusPlugIn 原理](https://blog.lindexi.com/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html)

从 WPF 使用 PenIMC 在 WISPTIS 服务获取 RealTime Stylus 到 StylusPlugIn 收到消息或在 Touch 等事件收到消息，请看 [WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html)

这就是需要涉及整个 WPF 的命中测试以及触摸输入机制，这也就是从手指触摸到屏幕到 WPF 框架将信息给业务层的步骤。还请小伙伴阅读本文的链接博客，本文接下来来和小伙伴聊聊下半部分的逻辑

在业务层收到了触摸的信息，如何转换为笔迹对象？首先笔迹的本质绘制就是将输入的离散的点，绘制成为 Geometry 几何加入到 WPF 的渲染中

在 WPF 中提供了 Stroke 类用于协助以上计算，通过给 Stroke 输入离散的点，可以通过调用 Stroke 的 Draw 方法，向某个 DrawingContext 绘制出 Geometry 的笔迹内容。这部分的逻辑很简单，请看 [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html)

在绘制到某个 Visual 里面之后，需要将 Visual 加入到 WPF 的视觉树中，在 WPF 的渲染机制里面，将会依据视觉树上的元素的更改刷新视觉树的渲染内容。这部分细节请看 [WPF 渲染原理](https://blog.lindexi.com/post/WPF-%E6%B8%B2%E6%9F%93%E5%8E%9F%E7%90%86.html)

而此时离屏幕渲染依然还有一段路线，在 WPF 通过 MIL 层，给出 Geometry 的绘制原语之后，将会和 WPF 界面的其他元素，如按钮文字等等在渲染线程合成为 DirectX 渲染图元，交给 DirectX 底层执行绘制。而其实在进行输出渲染图元这个步骤就算是进入了 DirectX 渲染管线部分，后续渲染管线的工作请看 [细说图形学渲染管线](https://zhuanlan.zhihu.com/p/79183044)

但事实上不是 WPF 将绘制原语准备好之后，就会调用 绘制调用 Draw Call 指令，显卡就会进行工作，为了让整体效率最高，系统层或者说 DirectX 将会打包多个 Draw call 指令，一次交给 GPU 去渲染

而经过了渲染管线之后是否就能在屏幕上实际显示？其实不然，还需要经过 DWM 桌面窗口管理器的调度，将多个窗口的画面合成之后在交给显卡的缓冲区，等待屏幕刷新

这就是整个的步骤

从这个步骤了解上，可以理解 [高性能笔迹原理](https://blog.lindexi.com/post/%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94%E8%BF%B9%E5%8E%9F%E7%90%86.html) 的内容

那开发者端能控制的部分包括哪些？首先是获取触摸的逻辑，可以选择从上层的 Touch 或 Stylus 事件获取触摸消息，也可以选择从底层的 StylusPlugIn 获取，当然也可以选择[禁用实时触摸](https://blog.lindexi.com/post/WPF-%E7%A6%81%E7%94%A8%E5%AE%9E%E6%97%B6%E8%A7%A6%E6%91%B8.html)通过 WM_Touch 消息获取触摸。不过采用 WM_Touch 就需要使用 [WPF 模拟触摸设备](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87.html) 的方法

如果硬件触摸框是可以定制的，那么也可以通过 USB 读取 HID 的方式拿到触摸框原始信息

其次就是如何绘制笔迹的方式，此时可以利用 WPF 框架提供的笔迹绘制算法，在完全开源的 WPF 框架里面，可以看到有一个大文件夹很多代码用来实现一个看起来比较顺滑的笔迹。小伙伴也可以去抄 WPF 的源代码自己魔改

然后笔迹的绘制方式基本上可以选 Geometry 或 Image 的方式，加入到视觉树中，或者重绘已有位图的方式

接着在进入 DX 渲染管线部分，可以使用 WPF 的 Effect 机制，通过 HLSL 对画面显示进行优化，这部分属于像素着色器的知识

- [WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html)
- [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html)
- [Windows 的 Pen 协议](https://blog.lindexi.com/post/Windows-%E7%9A%84-Pen-%E5%8D%8F%E8%AE%AE.html)
- [WPF 渲染原理](https://lindexi.gitee.io/post/WPF-%E6%B8%B2%E6%9F%93%E5%8E%9F%E7%90%86.html )
- [高性能笔迹原理](https://blog.lindexi.com/post/%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94%E8%BF%B9%E5%8E%9F%E7%90%86.html)
- [WPF 高性能笔](https://blog.lindexi.com/post/WPF-%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94.html ) 
- [WPF 高速书写 StylusPlugIn 原理](https://blog.lindexi.com/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html )
- [WPF 最小的代码使用 DynamicRenderer 书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E5%B0%8F%E7%9A%84%E4%BB%A3%E7%A0%81%E4%BD%BF%E7%94%A8-DynamicRenderer-%E4%B9%A6%E5%86%99.html )
- [WPF 使用 Composition API 做高性能渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Composition-API-%E5%81%9A%E9%AB%98%E6%80%A7%E8%83%BD%E6%B8%B2%E6%9F%93.html )
- [WPF 使用 Win2d 渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Win2d-%E6%B8%B2%E6%9F%93.html )
- [win10 uwp win2d CanvasVirtualControl 与 CanvasAnimatedControl](https://blog.lindexi.com/post/win10-uwp-win2d-CanvasVirtualControl-%E4%B8%8E-CanvasAnimatedControl.html )
- [WPF 禁用实时触摸](https://blog.lindexi.com/post/WPF-%E7%A6%81%E7%94%A8%E5%AE%9E%E6%97%B6%E8%A7%A6%E6%91%B8.html)
- [WPF 模拟触摸设备](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87.html)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。