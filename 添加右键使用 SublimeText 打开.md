# 添加右键使用 SublimeText 打开

最近修改了系统，重新安装 SublimeText 但是在安装的时候忘记设置右键使用 SublimeText 打开，所以就需要写注册表。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->

<!-- 标签：SublimeText -->

可以复制下面的代码到一个记事本，然后保存为 `sublime_addright.reg`，双击打开就可以。

```csharp
Windows Registry Editor Version 5.00
[HKEY_CLASSES_ROOT\*\shell\SublimeText3]
@="用 SublimeText3 打开"
"Icon"="D:\\Program Files\\Sublime Text 3\\sublime_text.exe,0"

[HKEY_CLASSES_ROOT\*\shell\SublimeText3\command]
@="D:\\Program Files\\Sublime Text 3\\sublime_text.exe %1"


[HKEY_CLASSES_ROOT\Directory\shell\SublimeText3]
@="用 SublimeText3 打开"
"Icon"="D:\\Program Files\\Sublime Text 3\\sublime_text.exe,0"

[HKEY_CLASSES_ROOT\Directory\shell\SublimeText3\command]
@="D:\\Program Files\\Sublime Text 3\\sublime_text.exe %1"
```

请把路径修改你自己的SublimeText路径。

上面为什么说需要使用记事本打开而不是使用SublimeText，因为注册表一般都是使用GBK，如果使用SublimeText打开，保存的是Utf8，这时会发现乱码。

如果不小心使用SublimeText打开的，右击出现乱码，那么可以通过在注册表修改值。打开运行，输入`regedit`进入注册表，可以看到注册表可以输入路径，输入`HKEY_CLASSES_ROOT\*\shell\SublimeText3`修改值

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F201834175414.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
