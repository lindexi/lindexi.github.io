
# WPF 两个 Topmost 的窗口如何设置谁在最上方

我需要有两个层级最高的窗口，但是要求某个窗口在另一个的上方，同时这两个窗口在所有其他的应用程序窗口的上方

<!--more-->


<!-- CreateTime:6/17/2020 8:29:09 PM -->



需要用到 SetWindowLong 的 win32 方法的设置，假设我有 A 和 B 两个窗口，我需要让这两个窗口都是 Topmost 同时 A 在 B 的上方

大概逻辑如下



```csharp
            B.Loaded += (sender, args) =>
            {
                A.Topmost = false;
                A.Topmost = true;

                B.Dispatcher.InvokeAsync(() =>
                {
                    Win32.User32.SetWindowLongPtr(new WindowInteropHelper(A).Handle,
                        Win32.GetWindowLongFields.GWL_HWNDPARENT, new WindowInteropHelper(B).Handle);
                });
            };
```

为什么需要在 B 的 Loaded 之后，原因是为了窗口实际创建出来，拿到句柄

为什么还需要做延迟一拍设置，因为立刻设置可能和控件初始化冲突，如 Popup 刚好弹出，将不会收起。注意这是在一个特别复杂的项目里面才发现这个坑，我创建空白项目没有发现这个坑

这里的 SetWindowLongPtr 是 SetWindowLong 方法，只是封装了 x86 和 x64 的代码

关于 GWL_HWNDPARENT 的定义如下

```csharp
    public enum GetWindowLongFields
    {
      GWL_USERDATA = -21, // 0xFFFFFFEB
      GWL_EXSTYLE = -20, // 0xFFFFFFEC
      GWL_STYLE = -16, // 0xFFFFFFF0
      GWL_ID = -12, // 0xFFFFFFF4
      GWL_HWNDPARENT = -8, // 0xFFFFFFF8
      GWL_HINSTANCE = -6, // 0xFFFFFFFA
      GWL_WNDPROC = -4, // 0xFFFFFFFC
    }
```

设置两个窗口有关联和 WPF 的设置 Owner 几乎等价，只是这个 win32 方法可以在复杂项目也设置上去





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。