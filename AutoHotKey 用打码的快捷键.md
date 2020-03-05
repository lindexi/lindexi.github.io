# AutoHotKey 用打码的快捷键

本文告诉大家如何使用 AutoHotKey 将 `-` 键默认输入的时候是下划线，因为使用下划线在写代码的时候是用在私有字段，而 `-` 很少使用

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


我打码经常需要使用下划线`_`而下划线需要按`shift+-` 两个键，我找到[autohotkey](https://www.autohotkey.com/) 可以用来修改键盘，关于这个工具的中文请看 [http://ahkcn.github.io/docs/AutoHotkey.htm](http://ahkcn.github.io/docs/AutoHotkey.htm)

首先在[官网](https://www.autohotkey.com/) 下载安装

安装完成之后可以写 AutoHotKey 的脚本，脚本的格式是 `ahk` 也是纯文本

使用 SublimeText 创建一个文本，将文本后缀名修改为 `.ahk` 然后添加下面的代码就可以让`-` 键默认输入的时候是下划线

```
+_::send -{blind}{Shift}
^-::^-
-::_

```

上面的代码做法是将 `-` 换下划线，在按下 `shift+-` 输入 `-` 这样就可以在输入变量的时候不需要总是使用 shift 键

因为在 VisualStudio 中，快捷键 `ctrl+-` 是返回，所以使用 `^-::^-` 让原先的 `ctrl+-` 作为返回

使用 AutoHotKey 可以写出很多有趣的代码，推荐小伙伴这个脚本