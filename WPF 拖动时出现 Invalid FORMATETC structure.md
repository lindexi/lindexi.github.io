# WPF 拖动时出现 Invalid FORMATETC structure

如果在 WPF 需要支持一个东西可以拖动，那么可以使用 DragDrop ，但是使用这个之后就出现了异常

```csharp
System.Runtime.InteropServices.COMException was unhandled
Message: An exception of type 'System.Runtime.InteropServices.COMException' occurred in PresentationCore.dll and wasn't handled before a managed/native boundary
Additional information: Invalid FORMATETC-Structure (Exception HRESULT: 0x80040064 (DV_E_FORMATETC))
```

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->

<!-- csdn -->

如果需要拖动一个文字，那么可以使用下面代码

```csharp
            DataObject dataObject = new DataObject(DataFormats.Text, "hello");
                DragDrop.DoDragDrop(this, dataObject, DragDropEffects.Move);
```

这个在程序内拖动不会出现问题，但是如果在拖动在程序外，就会出现

```csharp
'System.Runtime.InteropServices.COMException' occurred in PresentationCore.dll
System.Runtime.InteropServices.COMException was unhandled
Message: An exception of type 'System.Runtime.InteropServices.COMException' occurred in PresentationCore.dll and wasn't handled before a managed/native boundary
Additional information: Invalid FORMATETC-Structure (Exception HRESULT: 0x80040064 (DV_E_FORMATETC))
```

这是正常的，因为这是 windows 的坑。如果拖动的程序无法把你的内容转换为他需要的，那么就会出现这个错误。

只需要忽略就好了

参见 https://stackoverflow.com/a/34092811/6116637

![](http://www.chinadaily.com.cn/china/images/attachement/jpg/site1/20090722/0013729e4ad90bd108253f.jpg )