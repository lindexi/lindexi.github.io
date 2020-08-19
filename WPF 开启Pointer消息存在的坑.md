# WPF 开启Pointer消息存在的坑

本文记录在 WPF 开启 Pointer 消息的坑

<!--more-->
<!-- CreateTime:2019/12/24 14:33:41 -->

<!-- 发布 -->

启用了Pointer之后，调用Textbox.Focus()，起不来屏幕键盘，必须点在它之上才行，触摸在它之上才行

默认 Pointer 消息是使用屏幕绝对坐标而不是窗口坐标

可能存在获取 Stylus 事件时触摸点不准，此时可以通过获取 Touch 代替