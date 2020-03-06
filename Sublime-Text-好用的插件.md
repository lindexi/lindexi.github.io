
# Sublime Text 好用的插件

本文告诉大家一些好用的 Sublime Text 插件

<!--more-->


<!-- CreateTime:2018/10/17 10:14:40 -->


## Git Gutter

在左边显示新建的行和修改的

请看下面的图片

![](https://user-images.githubusercontent.com/16542113/28744712-f80ea13e-7466-11e7-96ac-51f453fb22b6.gif )

[jisaacks/GitGutter: A Sublime Text 2/3 plugin to see git diff in gutter](https://github.com/jisaacks/GitGutter )

## 从资源管理器打开当前的文件

通过点击 Preferences 的 Key Bindings 可以打开另一个窗口，输入下面的代码保存关掉就可以使用热键 `ctrl+alt+e` 从资源管理器打开当前的文件

```csharp
{ "keys": ["ctrl+alt+e"], "command": "open_dir", 
    "args": {"dir": "$file_path", "file": "$file_name"} }
```

如果是一个行而且后面没有 json 代码就不需要在最后添加逗号




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。