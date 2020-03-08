# win10 uwp 获取窗口的坐标和宽度高度

本文告诉大家几个方法在 UWP 获取窗口的坐标和宽度高度

<!--more-->
<!-- CreateTime:2018/11/26 15:04:00 -->

<!-- csdn -->

<div id="toc"></div>

## 获取可视范围

获取窗口的可视大小

```csharp
Windows.UI.ViewManagement.ApplicationView.GetForCurrentView().VisibleBounds
```

## 获取当前窗口的坐标和宽度高度

```csharp
Window.Current.Bounds
```

## 获取最前窗口的范围

通过 Win32 的 Api 获取最前的窗口的范围

```csharp

IntPtr hWID = GetForegroundWindow();

Rect rect;

Rect* ptr = &rect;

GetWindowRect(GetForegroundWindow(), pAngle);

return rect;



    [DllImport("user32.dll", CharSet = CharSet.Ansi)]
    private static extern IntPtr GetForegroundWindow();

    [DllImport("user32.dll", CharSet = CharSet.Ansi)]
    private unsafe static extern Boolean GetWindowRect(IntPtr intPtr, Rect* lpRect);

private struct Rect
{
    public int left;
    public int top;
    public int right;
    public int bottom;
}
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
