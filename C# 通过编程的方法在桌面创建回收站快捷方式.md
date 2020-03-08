# C# 通过编程的方法在桌面创建回收站快捷方式

基本所有的桌面都会存在回收站快捷方式，如果想要多创建几个快捷方式，点开就是回收站，请看本文的方法

<!--more-->
<!-- CreateTime:2019/11/29 8:30:35 -->


在引用 Windows Script Host Object Model 这个 COM 方法之后可以使用下面代码在桌面创建 `Recycle Bin.lnk` 快捷方式，这个快捷方式可以打开回收站

```csharp
            object shDesktop = "Desktop";
            WshShell shell = new WshShell();
            string shortcutAddress = (string) shell.SpecialFolders.Item(ref shDesktop) + @"\Recycle Bin.lnk";
            IWshShortcut shortcut = (IWshShortcut) shell.CreateShortcut(shortcutAddress);
            shortcut.Description = "New shortcut for Recycle Bin";
            shortcut.Hotkey = "Ctrl+Shift+N";
            shortcut.IconLocation = @"C:\WINDOWS\System32\imageres.dll";
            shortcut.TargetPath = "::{645ff040-5081-101b-9f08-00aa002f954e}";
            shortcut.Save();
```

参见 [使用 C# 代码创建快捷方式文件 - walterlv](https://blog.walterlv.com/post/create-shortcut-file-using-csharp.html )

[C# 如何引用 WshShell 类](https://blog.lindexi.com/post/C-%E5%A6%82%E4%BD%95%E5%BC%95%E7%94%A8-WshShell-%E7%B1%BB.html )

[c# - Programmatically create a shortcut to the recycle bin or other special folders - Stack Overflow](https://stackoverflow.com/a/41825480/6116637 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
