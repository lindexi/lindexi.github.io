
# SublimeText 配置跳转回上个光标坐标

在 VisualStudio 可以通过 ctrl+- 的功能，跳转到上个光标所在的坐标。如在多个方法之间跳转，可以通过这个快捷键快速实现。在 SublimeText 可以在菜单的 Goto 里面找到 Jump Back 功能，这个功能就是对应 VisualStudio 的跳转回上个光标的功能，也就是向后定位功能

<!--more-->


<!-- 发布 -->

对应的 SublimeText 的快捷键是 `alt+-` 和 VS 不相同，可以在 SublimeText 的 Preferences 的 Key Bindings 里面添加下面代码修改快捷键

```csharp
    { "keys": ["ctrl+-"], "command": "jump_back" },
```

这个快捷键是向后跳转，而向后跳转之后想要向前跳可以使用 `alt+shift+-` 快捷键，这对应 VS 的 `ctrl+shift+-` 快捷键，可以通过下面代码配置和 VS 相同

```csharp
    { "keys": ["ctrl+shift+-"], "command": "jump_forward" },
```





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。