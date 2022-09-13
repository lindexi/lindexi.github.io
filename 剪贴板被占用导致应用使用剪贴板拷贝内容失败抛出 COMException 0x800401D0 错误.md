# 剪贴板被占用导致应用使用剪贴板拷贝内容失败抛出 COMException 0x800401D0 错误

本文记录某些软件，例如 向日葵远程控制 软件占用剪贴板，导致 WPF 应用使用剪贴板拷贝内容和设置剪贴板时，抛出 System.Runtime.InteropServices.COMException (0x800401D0): OpenClipboard 失败 (0x800401D0 (CLIPBRD_E_CANT_OPEN)) 异常

<!--more-->
<!-- CreateTime:2022/9/9 15:39:17 -->

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