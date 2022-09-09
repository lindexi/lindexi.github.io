
# 剪贴板被占用导致应用使用剪贴板拷贝内容失败抛出 COMException 0x800401D0 错误

本文记录某些软件，例如 向日葵远程控制 软件占用剪贴板，导致 WPF 应用使用剪贴板拷贝内容和设置剪贴板时，抛出 System.Runtime.InteropServices.COMException (0x800401D0): OpenClipboard 失败 (0x800401D0 (CLIPBRD_E_CANT_OPEN)) 异常

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

现象：

访问剪贴板，例如调用 System.Windows.Clipboard.SetText 方法，将会抛出入夏异常

```
 System.Runtime.InteropServices.COMException (0x800401D0): OpenClipboard 失败 (0x800401D0 (CLIPBRD_E_CANT_OPEN))

   at System.Windows.Clipboard.Flush()
   at System.Windows.Clipboard.CriticalSetDataObject(Object data, Boolean copy)
   at System.Windows.Clipboard.SetDataObject(Object data, Boolean copy)
   at System.Windows.Clipboard.SetDataInternal(String format, Object data)
   at System.Windows.Clipboard.SetText(String text, TextDataFormat format)
   at System.Windows.Clipboard.SetText(String text)
```

解决方法：

关闭占用剪贴板的应用，例如关闭占用剪贴板的向日葵远程控制软件




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。