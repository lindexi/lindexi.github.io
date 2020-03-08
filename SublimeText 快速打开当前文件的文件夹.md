# SublimeText 快速打开当前文件的文件夹

在使用 SublimeText 需要使用资源管理器打开当前的文件，但是没有官方的方法，需要设置快捷键

<!--more-->
<!-- CreateTime:2019/2/26 18:45:29 -->

<!-- csdn -->

点击 Preferences 的 Key bindings 打开编辑界面，打开之后可以看到一个 json 文件，这个文件就是所有可以可以使用的快捷键

这里只修改用户的文件，也就是 Default (Windows).sublime-keymap 文件

在里面添加一项，输入下面代码

```csharp
	{
	    "keys": ["ctrl+alt+e"],
	    "command": "open_dir", 
        "args": 
        {
            "dir": "$file_path", 
            "file": "$file_name"
        } 
     }
```

需要注意，对于 json 最后一项是不添加逗号的，也就是如果你的文件里面只有一个括号，里面就直接输入而不需要添加逗号

```csharp
[
    {
        "keys": ["ctrl+alt+e"],
	    "command": "open_dir", 
        "args": 
        {
            "dir": "$file_path", 
            "file": "$file_name"
        } 
     }
]

```

现在可以在需要使用资源管理器打开的文件按下 ctrl+alt+e 打开，打开了的资源管理器会自动选择包含这个文件的文件夹，同时选择这个文件

通过快捷键的方法可以快速打开 SublimeText 打开的文件的所在位置。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
