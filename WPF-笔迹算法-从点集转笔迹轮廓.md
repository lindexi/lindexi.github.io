
# WPF 笔迹算法 从点集转笔迹轮廓

本文将告诉大家一些笔迹算法，从用户输入的点集，即鼠标轨迹点或触摸轨迹点等，转换为一个可在界面绘制显示笔迹画面的基础数学算法。尽管本文标记的是 WPF 的笔迹算法，然而实际上本文更侧重基础数学计算，理论上可以适用于任何能够支持几何绘制的 UI 框架上，包括 UWP 或 WinUI 或 UNO 或 MAUI 或 Eto 等框架

<!--more-->


<!-- 草稿 -->





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。