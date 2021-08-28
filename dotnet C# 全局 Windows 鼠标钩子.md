# dotnet C# 全局 Windows 鼠标钩子

本文来告诉大家一个简单的方法实现全局的 鼠标钩子

<!--more-->
<!-- 发布 -->

实现封装方法十分简单，请看代码

```csharp
    public class MouseHookEventArgs : EventArgs
    {
        public bool Handle { get; set; }

        /// <inheritdoc />
        public MouseHookEventArgs(MouseMessages mouseMessage)
        {
            MouseMessage = mouseMessage;
        }

        public MouseMessages MouseMessage { get; }

        public enum MouseMessages
        {
            MouseDown,
            MouseMove,
            MouseUp,
        }
    }

    /// <summary>
    /// 鼠标钩子
    /// </summary>
    public static class MouseHook
    {
        private delegate IntPtr MouseProc(int nCode, IntPtr wParam, IntPtr lParam);
        private static MouseProc _proc = HookCallback;
        private static IntPtr _hookID = IntPtr.Zero;
        private const int WH_MOUSE_LL = 14;

        private enum MouseMessages
        {
            WM_MOUSEMOVE = 0x0200
        }

        [StructLayout(LayoutKind.Sequential)]
        private struct POINT
        {
            public int x;
            public int y;
        }

        [StructLayout(LayoutKind.Sequential)]
        private struct MSLLHOOKSTRUCT
        {
            private POINT pt;
            private uint mouseData;
            private uint flags;
            private uint time;
            private IntPtr dwExtraInfo;
        }

        [DllImport("user32.dll", CharSet = CharSet.Auto, SetLastError = true)]
        private static extern IntPtr SetWindowsHookEx(int idHook,
            MouseProc lpfn, IntPtr hMod, uint dwThreadId);

        [DllImport("user32.dll", CharSet = CharSet.Auto, SetLastError = true)]
        [return: MarshalAs(UnmanagedType.Bool)]
        private static extern bool UnhookWindowsHookEx(IntPtr hhk);

        [DllImport("user32.dll", CharSet = CharSet.Auto, SetLastError = true)]
        private static extern IntPtr CallNextHookEx(IntPtr hhk, int nCode,
            IntPtr wParam, IntPtr lParam);

        /// <summary>
        /// 开启全局钩子
        /// </summary>
        /// <param name="moduleName"></param>
        public static void Start(string moduleName)
        {
            Debug.WriteLine($"模块 {moduleName} 开启全局鼠标钩子");

            _hookID = SetHook(_proc);
        }

        public static void Stop()
        {
            UnhookWindowsHookEx(_hookID);
        }

        private static IntPtr SetHook(MouseProc proc)
        {
            using (Process curProcess = Process.GetCurrentProcess())
            using (ProcessModule curModule = curProcess.MainModule)
            {
                return SetWindowsHookEx(WH_MOUSE_LL, proc,
                    Kernel32.GetModuleHandle(curModule.ModuleName), 0);
            }
        }
        private static IntPtr HookCallback(int nCode, IntPtr wParam, IntPtr lParam)
        {
            if (nCode >= 0 )
            {
                MouseHookEventArgs mouseHookEventArgs = null;
                switch ((WM) wParam)
                {
                    case WM.MOUSEMOVE:
                        mouseHookEventArgs=(new MouseHookEventArgs(MouseHookEventArgs.MouseMessages.MouseMove));
                        break;
                    case WM.LBUTTONDOWN:
                        mouseHookEventArgs=(new MouseHookEventArgs(MouseHookEventArgs.MouseMessages.MouseDown));
                        break;
                    case WM.LBUTTONUP:
                        mouseHookEventArgs=(new MouseHookEventArgs(MouseHookEventArgs.MouseMessages.MouseUp));
                        break;
                }
                //MSLLHOOKSTRUCT hookStruct = (MSLLHOOKSTRUCT)Marshal.PtrToStructure(lParam, typeof(MSLLHOOKSTRUCT));

                if (mouseHookEventArgs != null)
                {
                    OnMouseEvent(mouseHookEventArgs);

                    if (mouseHookEventArgs.Handle)
                    {
                        return IntPtr.Zero;
                    }
                }

            }
            return CallNextHookEx(_hookID, nCode, wParam, lParam);
        }

        public static event EventHandler<MouseHookEventArgs> MouseEvent;

        private static void OnMouseEvent(MouseHookEventArgs e)
        {
            MouseEvent?.Invoke(null, e);
        }
    }
```


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
