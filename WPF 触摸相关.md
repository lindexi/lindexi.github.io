# WPF 触摸相关

本文整理我写的触摸博客

<!--more-->
<!-- CreateTime:5/29/2020 4:56:25 PM -->

<!-- 发布 -->

## 框架和原理

[WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html )

[WPF 通过 InputManager 模拟调度触摸事件](https://blog.lindexi.com/post/WPF-%E9%80%9A%E8%BF%87-InputManager-%E6%A8%A1%E6%8B%9F%E8%B0%83%E5%BA%A6%E8%A7%A6%E6%91%B8%E4%BA%8B%E4%BB%B6.html )

[WPF 从触摸消息转触摸事件](https://blog.lindexi.com/post/WPF-%E4%BB%8E%E8%A7%A6%E6%91%B8%E6%B6%88%E6%81%AF%E8%BD%AC%E8%A7%A6%E6%91%B8%E4%BA%8B%E4%BB%B6.html )

[WPF 触摸底层 PenImc 是如何工作的](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%BA%95%E5%B1%82-PenImc-%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84.html )

## 功能

[win10 支持默认把触摸提升 Pointer 消息](https://blog.lindexi.com/post/win10-%E6%94%AF%E6%8C%81%E9%BB%98%E8%AE%A4%E6%8A%8A%E8%A7%A6%E6%91%B8%E6%8F%90%E5%8D%87-Pointer-%E6%B6%88%E6%81%AF.html )

[WPF 非客户区的触摸和鼠标点击响应](https://blog.lindexi.com/post/WPF-%E9%9D%9E%E5%AE%A2%E6%88%B7%E5%8C%BA%E7%9A%84%E8%A7%A6%E6%91%B8%E5%92%8C%E9%BC%A0%E6%A0%87%E7%82%B9%E5%87%BB%E5%93%8D%E5%BA%94.html )

[WPF 开启 ScrollViewer 的触摸滚动](https://blog.lindexi.com/post/WPF-%E5%BC%80%E5%90%AF-ScrollViewer-%E7%9A%84%E8%A7%A6%E6%91%B8%E6%BB%9A%E5%8A%A8.html )

[WPF 获得触摸精度和触摸点](https://blog.lindexi.com/post/WPF-%E8%8E%B7%E5%BE%97%E8%A7%A6%E6%91%B8%E7%B2%BE%E5%BA%A6%E5%92%8C%E8%A7%A6%E6%91%B8%E7%82%B9.html )

[WPF 禁用实时触摸](https://blog.lindexi.com/post/WPF-%E7%A6%81%E7%94%A8%E5%AE%9E%E6%97%B6%E8%A7%A6%E6%91%B8.html )

## 高级用法

[WPF 模拟触摸设备](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87.html )

[WPF 测试触摸设备发送触摸按下和抬起不成对](https://blog.lindexi.com/post/WPF-%E6%B5%8B%E8%AF%95%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87%E5%8F%91%E9%80%81%E8%A7%A6%E6%91%B8%E6%8C%89%E4%B8%8B%E5%92%8C%E6%8A%AC%E8%B5%B7%E4%B8%8D%E6%88%90%E5%AF%B9.html )

## 历史

[浅谈 Windows 桌面端触摸架构演进](https://blog.lindexi.com/post/%E6%B5%85%E8%B0%88-Windows-%E6%A1%8C%E9%9D%A2%E7%AB%AF%E8%A7%A6%E6%91%B8%E6%9E%B6%E6%9E%84%E6%BC%94%E8%BF%9B.html )

## 已知问题

[WPF 客户端开发需要知道的触摸失效问题](https://blog.lindexi.com/post/WPF-%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BC%80%E5%8F%91%E9%9C%80%E8%A6%81%E7%9F%A5%E9%81%93%E7%9A%84%E8%A7%A6%E6%91%B8%E5%A4%B1%E6%95%88%E9%97%AE%E9%A2%98.html )

[WPF 在触摸线程等待主线程窗口关闭会让主线程和触摸线程相互等待](https://blog.lindexi.com/post/WPF-%E5%9C%A8%E8%A7%A6%E6%91%B8%E7%BA%BF%E7%A8%8B%E7%AD%89%E5%BE%85%E4%B8%BB%E7%BA%BF%E7%A8%8B%E7%AA%97%E5%8F%A3%E5%85%B3%E9%97%AD%E4%BC%9A%E8%AE%A9%E4%B8%BB%E7%BA%BF%E7%A8%8B%E5%92%8C%E8%A7%A6%E6%91%B8%E7%BA%BF%E7%A8%8B%E7%9B%B8%E4%BA%92%E7%AD%89%E5%BE%85.html )

[WPF 开机启动因为触摸初始化锁住界面显示](https://blog.lindexi.com/post/WPF-%E5%BC%80%E6%9C%BA%E5%90%AF%E5%8A%A8%E5%9B%A0%E4%B8%BA%E8%A7%A6%E6%91%B8%E5%88%9D%E5%A7%8B%E5%8C%96%E9%94%81%E4%BD%8F%E7%95%8C%E9%9D%A2%E6%98%BE%E7%A4%BA.html )

[WPF 设置 WS_EX_TRANSPARENT 触摸失效](https://blog.lindexi.com/post/WPF-%E8%AE%BE%E7%BD%AE-WS_EX_TRANSPARENT-%E8%A7%A6%E6%91%B8%E5%A4%B1%E6%95%88.html )

![](http://image.acmx.xyz/lindexi%2F72551177_p0.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
