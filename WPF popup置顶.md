在程序写一个popup发现他会在置顶，在网上找了两大神代码，就使用他的代码

<!--more-->

<div id="toc"></div>

http://www.cnblogs.com/Leaco/p/3164394.html

http://blog.csdn.net/baijinwen/article/details/6159043

只需要把下面的类放到自己的软件，然后把使用 popup 替换为 CCPopup 就不会让popup置顶

```csharp
    public class CCPopup : Popup
    {
        public static DependencyProperty TopmostProperty = Window.TopmostProperty.AddOwner(typeof(CCPopup), new FrameworkPropertyMetadata(false, OnTopmostChanged));
        public bool Topmost
        {
            get { return (bool)GetValue(TopmostProperty); }
            set { SetValue(TopmostProperty, value); }
        }
        private static void OnTopmostChanged(DependencyObject obj, DependencyPropertyChangedEventArgs e)
        {
            (obj as CCPopup).UpdateWindow();
        }
        protected override void OnOpened(EventArgs e)
        {
            UpdateWindow();
        }
        private void UpdateWindow()
        {
            var hwnd = ((HwndSource)PresentationSource.FromVisual(this.Child)).Handle;
            RECT rect;
            if (GetWindowRect(hwnd, out rect))
            {
                SetWindowPos(hwnd, Topmost ? -1 : -2, rect.Left, rect.Top, (int)this.Width, (int)this.Height, 0);
            }
        }
        #region P/Invoke imports & definitions
        [StructLayout(LayoutKind.Sequential)]
        public struct RECT
        {
            public int Left;
            public int Top;
            public int Right;
            public int Bottom;
        }
        [DllImport("user32.dll")]
        [return: MarshalAs(UnmanagedType.Bool)]
        private static extern bool GetWindowRect(IntPtr hWnd, out RECT lpRect);
        [DllImport("user32", EntryPoint = "SetWindowPos")]
        private static extern int SetWindowPos(IntPtr hWnd, int hwndInsertAfter, int x, int y, int cx, int cy, int wFlags);
        #endregion
    }
    
```


## 另外的已知问题

[WPF 弹出 popup 里面的 TextBox 无法输入汉字 ](https://lindexi.github.io/lindexi/post/WPF-%E5%BC%B9%E5%87%BA-popup-%E9%87%8C%E9%9D%A2%E7%9A%84-TextBox-%E6%97%A0%E6%B3%95%E8%BE%93%E5%85%A5%E6%B1%89%E5%AD%97.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 