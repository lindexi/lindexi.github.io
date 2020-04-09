
# 从 DX 层面讲 WPF 渲染卡顿

这不是一篇深入底层的博客，很多细节还请看 DX 底层相关

<!--more-->


<!-- 发布 -->

小伙伴都知道 在 WPF 里面使用了 DX 作为底层的渲染，在说到 WPF 卡顿的时候，还请小伙伴不要忘记 dx 部分也是可能存在卡顿的

在 WPF 的 OnRender 方法调用完成之后，会让 dx 做什么？此时的 WPF 将会完成完整帧的帧绘制命令的收集，此时也是 dx 的完整帧的帧绘制命令的完成。然后 dx 将会传递这些绘制命令到 UMD (User Mode Driver) 层

上面这句话仅在单 UI 线程时生效，如果采用多 UI 线程将会复杂一些，本文也不讨论多 UI 线程

而在 WPF 的 OnRender 方法完成之后，其实只是将帧绘制命令传递到 UMD 而不是在屏幕显示

在 UMD 的功能是负责将收集的绘制命令转换为 GPU 能处理的工作批次，也就是 work batches 和命令缓冲器（Display Lists） 都是会根据对应的硬件 GPU 转换为不同的指令。这部分相对复杂，详细请看[官方文档](https://docs.microsoft.com/zh-cn/windows-hardware/drivers/display/)

在 UMD 完成之后，将会传递命令缓冲器回 D3D 让它将命令交给上下文队列，而KMD（ kernel mode GPU driver）层根据命令进行绘制，详细请看 [GPU Rendering Pipeline——GPU渲染流水线简介 - 知乎](https://zhuanlan.zhihu.com/p/61949898 )

绘制完成之后将会在 GPU 缓存里面绘制出一帧完整的图像，而此时依然还不是在屏幕显示，需要等待 Present 命令才会让屏幕输出

通过上文，当然需要您读一下附加的博客，如果 WPF 的 OnRender 卡顿了，此时没有输出绘制命令到 DX 那么将会让完整帧的帧绘制命令延迟，这部分都在用户代码上，比较好调试

第二部分是在收集到的绘制命令转对应的绘制指令，这部分和具体的设备相关，如果给了一些有毒的绘制，那么转换时间将会比较长。换句话说是驱动程序需要使用比预期长的时间才能为GPU准备好需要渲染的某一帧，这部分会在 win7 的切换 dpi 不重启时，部分硬件设备复现

第三部分就是虽然我的绘制命令很少，但是这个绘制命令是一个大几何，刚好这个几何处理是 GPU 没有优化的，那么 GPU 需要使用超过比较长的时间。此部分的 GPU 设备在高端用户很少会遇到，但是我刚好是开发面向大量古老设备的应用，意味着我在使用 PathGeometry 时需要关注这个几何的大小

而根据垂直刷新，只要错过了这帧，将会在下一次绘制才会输出。也就是延迟 1ms - 8ms 对于延迟来说基本相同，也就是一次性能优化，至少需要优化超过 8ms 否则很少有效果

这里插入一点的是 WIndows 系统不是只有一个应用在绘制，也就是在存在大量 CPU 的时候，将会让 GPU 的工作线程的运行被频繁中断，在一次渲染过程中存在大量线程的变化。换句话说，一个高性能的 UI 框架反而不是线程越多越好。这部分还请看 DirectX 多线程渲染性能相关

[一篇文章入门DirectX 12 - 知乎](https://zhuanlan.zhihu.com/p/57061190 )

[提交命令缓冲区 - Windows drivers | Microsoft Docs](https://docs.microsoft.com/zh-cn/windows-hardware/drivers/display/submitting-a-command-buffer )

[GPU Rendering Pipeline——GPU渲染流水线简介 - 知乎](https://zhuanlan.zhihu.com/p/61949898 )

本文的信息比较密集，我能讲的只是一个大概，更多还请小伙伴看本文引用的链接，特别是官方文档

我认为我写的内容最多只能算官方文档的笔记，唯一有点用的就是和 WPF 的关联

在 WPF 中，从 dx 层面出现的卡顿调试顺序建议如下

- 在 OnRender 的执行时间以及频率，通过 ContentRender 事件可以拿到频率。但是这个事件仅在调试下使用，同时监听此事件将会降低渲染性能
- 在 WPF 里面使用用到复杂的文本或几何
- 是否在 WPF 中开启大量的 UI 线程

另外，有一点需要注意，在调试渲染性能的时候，本身调试就会影响卡顿。换句话说用调试方式测量是在哪卡顿是不准确的，在渲染卡顿里面基本上就是慢1ms就是卡顿，而快7ms还没有优化

如果在没有找到 WPF 层的问题，而想要了解是否 dx 的渲染卡顿，可以尝试使用 Fraps 工具





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。