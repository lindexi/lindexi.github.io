# 快捷键

现在 UWP 没有快捷键，没有和 WPF 那样的快捷键，所以需要自己去写一个。

虽然可以用 Alt 的方法做出快捷键，但是需要做出特殊的，如 自定一个使用 ctrl 的快捷键在 UWP 是没有直接的做法。所以如何去使用比较简单的方法做自己的快捷键？

本文提供一个新的方法，可以做出快捷键。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>
<!-- csdn -->
<!-- 不发布 -->

现在 UWP 提供的 Alt 方法是 `AcceleratorKey` 使用他可以做出快捷键，但是功能比 WPF 很少。下面告诉大家如何使用这个类。

本文的这个方法来着 [【Windows 10 应用开发】使用快捷访问键 - 东邪独孤 - 博客园](http://www.cnblogs.com/tcjiaan/p/7019369.html) 再次表示感谢。


## 定义自己的快捷键

本文说的自定义快捷键不是来自 [【Windows 10 应用开发】自定义快捷键 - 东邪独孤 - 博客园](http://www.cnblogs.com/tcjiaan/p/7047754.html) ，本文的方法是通过简单的后台代码来做。



快捷键参见：https://github.com/Microsoft/Windows-universal-samples/blob/6370138b150ca8a34ff86de376ab6408c5587f5d/Samples/XamlUIBasics/cs/AppUIBasics/Common/NavigationHelper.cs

参见：https://msdn.microsoft.com/en-us/library/windows/apps/xaml/Hh868161(v=win.10).aspx#keyboard_shortcuts

https://msdn.microsoft.com/en-us/library/windows/apps/xaml/hh868246.aspx

快捷键需要业界一样，不能自己做标准，[Keyboard shortcuts in Windows - Windows Help](https://support.microsoft.com/en-us/help/12445/windows-keyboard-shortcuts)

原文：http://juniperphoton.net/2015/12/04/how-to-achieve-global-keyboard-shortcuts-in-uwp-apps-2/

https://github.com/Windows-XAML/Template10/blob/64c419acd55b875f25812c44ff131af12430eb34/Template10%20(Library)/Behaviors/KeyBehavior.cs


