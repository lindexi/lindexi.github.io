# Windows 对全屏应用的优化

全屏应用对应的是窗口模式应用，全屏应用指的是整个屏幕都是被咱一个应用独占了，屏幕上没有显示其他的应用，此时的应用就叫全屏应用。如希沃白板这个程序。本文主要告诉大家从微软官方的文档以及考古了解到的 Windows 对全屏应用的优化，以及是如何进行的优化，方便小伙伴在撕的时候可以找到根据

<!--more-->
<!-- CreateTime:5/1/2020 9:24:46 AM -->



当然，很多小伙伴只是需要依据，所以我就先贴出一篇特别好的官方文档，当然本文大部分内容都是从这篇文档抄的

- 解密Windows对全屏的优化： [Demystifying Fullscreen Optimizations](https://devblogs.microsoft.com/directx/demystifying-full-screen-optimizations/ )
- [Direct3D 9Ex Improvements - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/direct3darticles/direct3d-9ex-improvements )
- [For best performance, use DXGI flip model](https://devblogs.microsoft.com/directx/dxgi-flip-model/ )
- [Windowed, Fullscreen, and Borderless Modes: Which One Is Best?](https://www.makeuseof.com/tag/windowed-fullscreen-borderless-modes/ )
- [fullscreen mode and windowed mode_网络_安柏霖的专栏-CSDN博客](https://blog.csdn.net/toughbro/article/details/82926100 ) 

在开始聊 Windows 对全屏应用的优化之前，需要先聊聊窗口的显示方式。在 Windows 上运行的应用，如游戏等有三个不同的显示模式（考古发现还有一些特别的显示模式，但是太诡异了我就不敢说了）包括 全屏独占模式（FSE Fullscreen Exclusive）、窗口模式（Windowed）、无边框窗口。全屏独占窗口可以让应用独占显示和拥有更多（不是全部占有，但也差不多）的显卡资源。而在进入窗口模式的应用需要和其他的应用共享显示和计算资源，其他窗口在后台运行的依然需要使用显卡计算资源。此时的窗口模式应用需要依靠桌面管理器（DWM Desktop Window Manager）进行调度，此时的显卡资源也就需要将资源共享给其他所有的应用，而不是和独占全屏一样。而第三个模式无边框模式的窗口，应用依然是窗口运行，但是这个窗口没有边框，这意味着窗口可以调整大小用来做全屏，但此时的其他应用依然在后台运行

在微软的这篇文档  [Demystifying Fullscreen Optimizations](https://devblogs.microsoft.com/directx/demystifying-full-screen-optimizations/ ) 说在 win10 发布之后，微软进行了全屏窗口优化。但是考古发现其实在 win7 的 dx9 就有这样的优化，详细请看  [Direct3D 9Ex Improvements - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/direct3darticles/direct3d-9ex-improvements ) 官方文档

在微软官方文档说 win10 的一个优化是对无边框窗口进行全屏显示的应用的优化，对于无边框的窗口可以让用户体验和全屏独占应用一样的方法，但是会比独占全屏模式更加有利。原因在于独占全屏模式会出现切换的等待，虽然在大部分设备上有显卡的优化，这部分的时间非常短，但是如果想要做频繁的切换，效果也不是很好。而微软 win10 的优化另一部分就是对提供了更快的应用命令，如 alt+tab 切换窗口和多个屏幕的显示效果。通过大量的测试微软对全屏的无边框窗口的性能优化基本达到了全屏独占窗口的性能

什么全屏独占窗口能具有更高的性能？其实在上文有提到的是可以独占所有的显示，也就是屏幕的每个像素都是由这个应用控制的，此时的显卡可以使用更多的计算资源给到这个应用。但是为什么后面又提出了让无边框窗口通过修改大小做到全屏？然后花大量的资源优化无边框全屏窗口的性能。原因在于以下：

- 全屏独占应用在分辨率切换的时候的处理相对复杂，有大量的应用没有对这方面进行支持
- 全屏独占应用的显卡支持也是需要具体显卡的
- 如果有需求让其他的窗口，如游戏工具栏，如 xbox 游戏工具栏覆盖在全屏独占窗口时，就需要拦截全屏独占窗口的渲染。这个拦截会出现比较多性能问题和不稳定问题
- 全屏独占窗口对于多个屏幕的设备不够友好，如果使用 DWM 管理另一个屏幕，那么当另一个屏幕的应用获取焦点又需要如何处理。在 windows 的几个版本里面的处理是在其他窗口获取焦点的时候自动最小化全屏独占窗口，这样就让玩全屏独占应用的小伙伴很难同时一边玩全屏游戏一边看电影
- 上文说到的如果是窗口模式，那么渲染此时由 DWM 做的，但是如果有应用进入全屏独占模式，此时 DWM 需要将显示交给应用，此时会出现屏幕切换。如果进行频繁的切换如 alt+tab 那么这个效果比较差

此时小伙伴应该就能了解到为什么微软对全屏应用的优化将不仅是对全屏独占应用的优化，还包括对无边框窗口的全屏显示优化了。其实无边框窗口本身也是一个窗口，此时的屏幕依然是 DWM 管理显示

在 windows xp 开始，在有硬件设备的支持下，微软能给全屏独占应用特别好的支持，可以让几乎所有的显卡资源用在了全屏独占应用，同时停掉后台运行窗口的实际渲染（古老渲染方式的窗口请忽略）而在没有硬件设备的支持下，微软也能做到让计算资源大部分放在全屏独占应用

在 Windows 10 开始（我记得 win7 dx9 就有这个功能）微软不仅可以对全屏独占应用提供性能优化支持，同时对无边框的全屏窗口提供几乎同等的性能优化，此时更多的应用都选择使用无边框的全屏窗口而不是全屏独占窗口

但如果是进行更多的性能优化，可以考虑进入全屏独占窗口

如果是 WPF 程序，那么设置无边框然后设置窗口大小和屏幕一样大，也能在 win10 下拿到这部分优化效果。更多关于 WPF 的优化请看 [从 DX 层面讲 WPF 渲染卡顿](https://blog.lindexi.com/post/%E4%BB%8E-DX-%E5%B1%82%E9%9D%A2%E8%AE%B2-WPF-%E6%B8%B2%E6%9F%93%E5%8D%A1%E9%A1%BF.html )

注：有小伙伴对于独占全屏应用的多屏幕处理觉得有点迷，我换句话告诉大家，请打开一个 UWP 的视频应用，让他进入全屏。这个应用进入全屏就是独占模式，此时你在另一个屏幕移动一个窗口，逐步移动到视频应用的屏幕上，你可以看到要么视频的屏幕依然播放视频，要么就是你移动过去了，视频应用就最小化了

另外在 win7 下的对无边框的全屏应用的优化也是有坑的，要求你的应用需要覆盖屏幕的每一个像素才会隐藏任务栏。在一些有趣的设备上，不会给无边框的全屏应用设置前台焦点，因为窗口没有焦点所以此时的任务栏依然可以显示，解决方法请看 [WPF 让窗口激活作为前台最上层窗口的方法](https://blog.lindexi.com/post/WPF-%E8%AE%A9%E7%AA%97%E5%8F%A3%E6%BF%80%E6%B4%BB%E4%BD%9C%E4%B8%BA%E5%89%8D%E5%8F%B0%E6%9C%80%E4%B8%8A%E5%B1%82%E7%AA%97%E5%8F%A3%E7%9A%84%E6%96%B9%E6%B3%95.html)

更多博客

[Windows桌面实现之七（DirectX HOOK 方式截取特殊的全屏程序之一）](https://blog.csdn.net/fanxiushu/article/details/89363222 )

[C# 纯控制台创建一个全屏窗口](https://blog.lindexi.com/post/C-%E7%BA%AF%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%85%A8%E5%B1%8F%E7%AA%97%E5%8F%A3.html)

[WPF 全屏透明窗口](https://blog.lindexi.com/post/WPF-%E5%85%A8%E5%B1%8F%E9%80%8F%E6%98%8E%E7%AA%97%E5%8F%A3.html)


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
