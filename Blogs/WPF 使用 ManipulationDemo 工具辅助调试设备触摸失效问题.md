本文将和大家介绍我所在的团队开源的 ManipulationDemo 工具。通过 ManipulationDemo 工具可以提升调试设备触摸失效的效率

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

此工具在 GitHub 上完全开源，请看 [https://github.com/dotnet-campus/ManipulationDemo/](https://github.com/dotnet-campus/ManipulationDemo/)

软件界面效果大概如下

<!-- ![](image/WPF 通过 InputManager 模拟调度触摸事件/WPF 通过 InputManager 模拟调度触摸事件.gif) -->

![](http://image.acmx.xyz/lindexi%2FWPF%2520%25E9%2580%259A%25E8%25BF%2587%2520InputManager%2520%25E6%25A8%25A1%25E6%258B%259F%25E8%25B0%2583%25E5%25BA%25A6%25E8%25A7%25A6%25E6%2591%25B8%25E4%25BA%258B%25E4%25BB%25B6.gif)

可以显示接收到的 Win32 消息、当前的触摸设备状态，以及设备插拔信息

使用此工具可以用来调试设备触摸失效问题，包括：

- 是否有应用收走了消息
  - 通过查看应用左侧 Win32 消息即可了解。常见问题是有透明窗口挡住应用
- 是否有奇怪的 Win32 消息发过来导致触摸失效
- 是否出现设备插拔，导致触摸设备被拔走
  - 当出现 USB 设备插拔时，将会在应用下方的中间记录变更的设备信息，此时一般关注的是设备的 VID 和 PID 属性，从而了解是否自己的触摸框被插拔
  - 自己的应用使用的是 .NET Framework 低版本且用户端运行时也是 .NET Framework 低版本时，存在插拔触摸失效已知问题。详细请看 [WPF 插拔触摸设备触摸失效](https://blog.lindexi.com/post/WPF-%E6%8F%92%E6%8B%94%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87%E8%A7%A6%E6%91%B8%E5%A4%B1%E6%95%88.html )
  - 现在存在某些厂商的触摸框，进行插拔之后会导致触摸失效
- 是否触摸设备本身状态异常，即获取不到正确的触摸信息
  - 触摸点数拿不到或触摸设备数拿不到，从应用右下角可以看到正常触摸设备会显示触摸点数等
- 是否出现了多指触摸失效问题，即触摸转鼠标问题
  - 多指触摸只收到鼠标消息，且多指触摸的时候，界面没有反映
- 是否自己应用上层业务逻辑编写不正确
  - 即自己的应用触摸失效了，但 ManipulationDemo 工具可以正常收到触摸消息，这就证明不是设备的问题

更多触摸和笔迹书写相关请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html)
