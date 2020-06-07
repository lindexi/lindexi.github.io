# WPF 获取某个窗口的所有子窗口

如果不用到 Win32 方法，可以尝试遍历所有窗口获取 Owner 判断，不过此方法仅仅适合只有一个主线程

<!--more-->
<!-- CreateTime:6/6/2020 8:51:21 AM -->

<!-- 发布 -->

通过 Win32 的 EnumChildWindows 可以拿到某个窗口句柄的所有子窗口，大概用法如下，先定义一个辅助类

```csharp
public class WindowHandleInfo
{
    public static List<IntPtr> GetAllChildHandles(IntPtr handle)
    {
        List<IntPtr> childHandles = new List<IntPtr>();
 
        GCHandle gcChildhandlesList = GCHandle.Alloc(childHandles);
        IntPtr pointerChildHandlesList = GCHandle.ToIntPtr(gcChildhandlesList);
 
        try
        {
            EnumWindowProc childProc = new EnumWindowProc(EnumWindow);
            EnumChildWindows(handle, childProc, pointerChildHandlesList);
        }
        finally
        {
            gcChildhandlesList.Free();
        }
 
        return childHandles;
    }

    private delegate bool EnumWindowProc(IntPtr hwnd, IntPtr lParam);
 
    [DllImport("user32")]
    [return: MarshalAs(UnmanagedType.Bool)]
    private static extern bool EnumChildWindows(IntPtr window, EnumWindowProc callback, IntPtr lParam);
 
    private static bool EnumWindow(IntPtr hWnd, IntPtr lParam)
    {
        GCHandle gcChildhandlesList = GCHandle.FromIntPtr(lParam);
 
        if (gcChildhandlesList == null || gcChildhandlesList.Target == null)
        {
            return false;
        }
 
        List<IntPtr> childHandles = (List<IntPtr>) gcChildhandlesList.Target;
        childHandles.Add(hWnd);
 
        return true;
    }
}
```

使用方法是先拿到窗口的句柄，然后传入 GetAllChildHandles 方法，就可以拿到所有子窗口的句柄

```csharp
            var windowInteropHelper = new WindowInteropHelper(window);
            var hwnd = windowInteropHelper.Handle;
            var childWindowList = WindowHandleInfo.GetAllChildHandles(hwnd);
```

[PINVOKE: Getting all child handles of window · Software adventures and thoughts](http://blog.ralch.com/2015/04/pinvoke-getting-all-child-handles-of-window/ )

[使用 EnumWindows 找到满足你要求的窗口 - walterlv](https://blog.walterlv.com/post/find-specific-window-by-enum-windows.html )


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
