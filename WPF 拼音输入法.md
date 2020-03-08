# WPF 拼音输入法

本文来告诉大家如何使用 WPF 来写一个输入法，使用的方式是钩子。

<!--more-->
<!-- CreateTime:2019/6/5 17:06:58 -->

<!-- csdn -->
<div id="toc"></div>
<!-- 标签：输入法，wpf -->

实际上本文是在使用一个好用的软件 [希沃白板](http://easinote.seewo.com/) 的时候发现在里面很难输入拼音来做课堂活动。

![](http://image.acmx.xyz/lindexi%2F2018430111201441.jpg)

因为现在没有找到一个软件可以用来输入拼音的，快速的输入。输入音调是比较难的，所以我就重新做了一个输入法。

在[yswenli](http://www.cnblogs.com/yswenli/ )的帮助，使用了[yswenli/Wenli.IEM](https://github.com/yswenli/Wenli.IEM ) 方法做了一个输入法。

## 键盘

如果要做一个输入法，可以使用很多方法，本文使用的是全局 hook 的方式，需要注意，这个方式很容易让 360 杀掉。

注册钩子的方法很简单，只需要一个函数

```csharp
        [DllImport("user32.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.StdCall)]
        public static extern int SetWindowsHookEx(int idHook, HookProc lpfn, IntPtr hInstance, int threadId);
```

关于 HookProc 请看代码

```csharp
        public delegate int HookProc(int nCode, Int32 wParam, IntPtr lParam);

```

所以简单的注册一个钩子只需要三行代码

```csharp
                var keyboardHookProcedure = new HookProc(KeyboardHookProc);

                var hKeyboardHook = SetWindowsHookEx(WH_KEYBOARD_LL, keyboardHookProcedure, GetModuleHandle(Process.GetCurrentProcess().MainModule.ModuleName), 0);

                private int KeyboardHookProc(int nCode, Int32 wParam, IntPtr lParam)
                {
                	// 如果你要算括号就有很多行
                }
```

那么拿到了 hook 可以如何使用，下面来告诉大家如何解析信息

## 解析键盘

解析的方式微软有说到，因为很简单，只需要定义一个结构，请看代码。定义的结构需要是这样的，不要去优化。

```csharp
        [StructLayout(LayoutKind.Sequential)]
        public class KeyboardHookStruct
        {
            public int vkCode;  //定一个虚拟键码。该代码必须有一个价值的范围1至254
            public int scanCode; // 指定的硬件扫描码的关键
            public int flags;  // 键标志
            public int time; // 指定的时间戳记的这个讯息
            public int dwExtraInfo; // 指定额外信息相关的信息
        }
```

从参数就可以拿到，因为参数是指针，需要`Marshal.PtrToStructure`来拿到

```csharp
 KeyboardHookStruct myKeyboardHookStruct = (KeyboardHookStruct) Marshal.PtrToStructure(lParam, typeof(KeyboardHookStruct));
```

现在通过这个方法就可以拿到键盘的输入。

## 获得按键

虽然已经解析了，但是现在还是不知道用户按键是哪个。需要通过下面的方法转换，首先引用 WinForm ，因为定义在 WinForm 有，而且下面发送消息也是需要通过。

右击引用，点击程序集、框架，就可以看到 System.Windows.Forms ，请看图片

![](http://image.acmx.xyz/lindexi%2F20184301122315730.jpg)

引用了之后就可以使用下面的方法拿到按键

```csharp
Keys keyData = (System.Windows.Forms.Keys) myKeyboardHookStruct.vkCode;
```

尝试点一下，是不是就可以看到对应的值？

有了按键，那么下面如何写一个输入法就是需要使用了对应的算法了，如果想使用微软提供的算法，请看[C# 输入法](http://www.cnblogs.com/yswenli/p/6528447.html )，我是需要用来输入拼音。所以下面来告诉大家如何从用户按键拿到用户想要的输入。

## 输入流向

虽然已经拿到了按键，但是拿到的按键还是需要转换字符串才可以处理

```csharp
var key = keyData.ToString().ToLower();
```

现在的 key 就是一个字符串，在输入拼音，用户想的是快速的输入，而不是不停复制粘贴，对于普通的字符输入是可以直接输入，但是对于`āáǎ`的输入就无法直接输入的。

我看到一位老师是在记事本写了下面代码

```csharp
āáǎà
ēéěè
īíǐì
ōóǒò
ūúǔù
ǖǘǚǜ
```

需要哪个就去复制哪个，如写 `yùn` 就需要输入 `y` 然后复制一下`ù`，然后写 `n` ，这样想输入连续的拼音是很慢的。

所以对于`a,e,i,o,u,v`才需要输入法转换，对于其他的就直接输入就好了。

![](http://image.acmx.xyz/lindexi%2F20184301135348021.jpg)

那么如何让用户的按键无法直接输入到对应的程序，就需要使用下面的函数

```csharp
        [DllImport("user32.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.StdCall)]
        public static extern int CallNextHookEx(int idHook, int nCode, Int32 wParam, IntPtr lParam);
```

使用这个函数就可以通过信息钩子继续下一个钩子，在 `private int KeyboardHookProc(int nCode, Int32 wParam, IntPtr lParam)` 大家也看到有一个返回值，通过这个返回值可以告诉系统，是不是要把这个消息传给下一个程序。

如果返回的是 0 那么就是告诉系统，这个 hook 不处理，你需要把消息发给其他的程序。如果返回不是0 ，那么就是告诉系统，这个我处理了，其他的程序不能收到。

输入法判断用户输入的是 [a,z] [0,9] 告诉系统，不要发给其他的程序。

那么如果用户输入的不是 `a,e,i,o,u,v` 也就是可以直接给其他程序，这时怎么做？

实际上不管用户输入的是什么，只要发给其他程序都需要使用这个方法

```csharp
System.Windows.Forms.SendKeys.SendWait(string str)
```

通过这个方法就会把 str 发送给当前用户输入的程序。

## 算法

现在可以拿到了全部的输入，而且知道了如何把转换的值发送给用户，大概一个输入法就是需要这两个。

如果从用户的输入知道用户需要的什么就是算法，下面使用的方法很简单。

判断用户输入的是不是`a,e,i,o,u,v`，发现不是就直接发送输入。如果是就不发送任何输入，让用户选需要的是哪个

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018116195923.jpg)

这是我把它显示出来，代码还需要一个属性，表示当前是不是存在用户没有选的字符。如果存在，下一次输入的是[0,9]就是用户选的字符，因为只要5个可以选，对于大于5和0的就直接发送输入。当前对于现在很多输入法，都是按空格选第一个，这里也是需要判断用户输入的是不是空格。

在开发的时候发现还有很多的细节，不过这些我就不在这里告诉大家，我把代码放在下面，大家看一下。

我把程序放在论坛，可以点击 [快速在课堂活动输入拼音](http://bbs.seewoedu.cn/forum.php?mod=viewthread&tid=10625&highlight=%E6%8B%BC%E9%9F%B3 ) 下载

源代码请看

```csharp
       public MainWindow()
        {
            InitializeComponent();
            Loaded += MainWindow_Loaded;
            DataContext = this;
            Topmost = true;
        }

        private void HideVexogzPybnj()
        {
            LdefkgzYclfeufwx.Visibility = Visibility.Visible;
            KqhRst.Visibility = Visibility.Hidden;
        }

        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            HideVexogzPybnj();

            App.KeyBordHook.KeyUpEvent += KeyBordHook_KeyUpEvent;
            App.KeyBordHook.OnSpaced += KeyBordHook_OnSpaced;
            App.KeyBordHook.OnBacked += KeyBordHook_OnBacked;
        }

        private void KeyBordHook_OnBacked()
        {
            if (!_yuanyin)
            {
                _key = "";
                App.KeyBordHook.IsStarted = false;
                return;
            }

            _yuanyin = false;
            HideVexogzPybnj();
        }

        private void KeyBordHook_OnSpaced(int choose)
        {
            if (choose > 0)
            {
                choose = choose - 1;
            }

            if (choose < 5)
            {
                if (_yuanyin)
                {
                    App.KeyBordHook.Send(_yuanyinList[_key][choose].ToString());
                    _yuanyin = false;
                    HideVexogzPybnj();
                    return;
                }
            }

            App.KeyBordHook.IsStarted = false;
        }

        private void KeyBordHook_KeyUpEvent(object sender, System.Windows.Forms.Keys e)
        {
            var key = e.ToString().ToLower();

            List<string> yuanyin = "a,e,i,o,u,v".Split(',').ToList();

            if (yuanyin.Any(temp => temp == key))
            {
                if (_yuanyin)
                {
                    App.KeyBordHook.Send(_key);
                }
                _key = key;
                ShowDvfarVawmkezbi(_yuanyinList[key]);
                _yuanyin = true;
            }
            else
            {
                if (_yuanyin && !string.IsNullOrEmpty(_key))
                {
                    _yuanyin = false;
                    App.KeyBordHook.Send(_yuanyinList[_key][4].ToString());
                    _key = "";
                    HideVexogzPybnj();
                }

                App.KeyBordHook.Send(key);
            }
        }


        private string _key;
        private bool _yuanyin;

        private void ShowDvfarVawmkezbi(string str)
        {
            StringBuilder temp = new StringBuilder();
            for (var i = 0; i < str.Length; i++)
            {
                temp.Append((i + 1) + "." + str[i] + "    ");
            }

            Key = temp.ToString();

            KqhRst.Visibility = Visibility.Visible;
            LdefkgzYclfeufwx.Visibility = Visibility.Hidden;
        }

        public static readonly DependencyProperty KeyProperty = DependencyProperty.Register(
            nameof(Key), typeof(string), typeof(MainWindow), new PropertyMetadata(default(string)));

        public string Key
        {
            get => (string) GetValue(KeyProperty);
            set => SetValue(KeyProperty, value);
        }

        private readonly Dictionary<string, string> _yuanyinList = new Dictionary<string, string>()
        {
            {"a","āáǎàa" },
            {"e","ēéěèe" },
            {"i","īíǐìi" },
            {"o","ōóǒòo" },
            {"u","ūúǔùu" },
            {"v","ǖǘǚǜü" },
        };
```

```csharp
    class KeyboardHook
    {

        #region win32
        public delegate int HookProc(int nCode, Int32 wParam, IntPtr lParam);
        static int hKeyboardHook = 0; //声明键盘钩子处理的初始值
        public const int WH_KEYBOARD_LL = 13;   //线程键盘钩子监听鼠标消息设为2，全局键盘监听鼠标消息设为13
        HookProc KeyboardHookProcedure; //声明KeyboardHookProcedure作为HookProc类型

        //键盘结构
        [StructLayout(LayoutKind.Sequential)]
        public class KeyboardHookStruct
        {
            public int vkCode;  //定一个虚拟键码。该代码必须有一个价值的范围1至254
            public int scanCode; // 指定的硬件扫描码的关键
            public int flags;  // 键标志
            public int time; // 指定的时间戳记的这个讯息
            public int dwExtraInfo; // 指定额外信息相关的信息
        }

        [StructLayout(LayoutKind.Sequential)]
        public struct INPUT
        {
            public InputType dwType;
            public KEYBDINPUT ki;
        }
        [StructLayout(LayoutKind.Sequential, Pack = 1)]
        public struct KEYBDINPUT
        {
            public Int16 wVk;
            public Int16 wScan;
            public KEYEVENTF dwFlags;
            public Int32 time;
            public IntPtr dwExtraInfo;
        }

        public enum KEYEVENTF
        {
            EXTENDEDKEY = 1,
            KEYUP = 2,
            UNICODE = 4,
            SCANCODE = 8,
        }

        //使用此功能，安装了一个钩子
        [DllImport("user32.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.StdCall)]
        public static extern int SetWindowsHookEx(int idHook, HookProc lpfn, IntPtr hInstance, int threadId);


        //调用此函数卸载钩子
        [DllImport("user32.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.StdCall)]
        public static extern bool UnhookWindowsHookEx(int idHook);


        //使用此功能，通过信息钩子继续下一个钩子
        [DllImport("user32.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.StdCall)]
        public static extern int CallNextHookEx(int idHook, int nCode, Int32 wParam, IntPtr lParam);


        // 取得当前线程编号（线程钩子需要用到）
        [DllImport("kernel32.dll")]
        static extern int GetCurrentThreadId();


        //使用WINDOWS API函数代替获取当前实例的函数,防止钩子失效
        [DllImport("kernel32.dll")]
        public static extern IntPtr GetModuleHandle(string name);




        private const int WM_KEYDOWN = 0x100;//KEYDOWN
        private const int WM_KEYUP = 0x101;//KEYUP
        private const int WM_SYSKEYDOWN = 0x104;//SYSKEYDOWN
        private const int WM_SYSKEYUP = 0x105;//SYSKEYUP

        //ToAscii职能的转换指定的虚拟键码和键盘状态的相应字符或字符
        [DllImport("user32")]
        public static extern int ToAscii(int uVirtKey, //[in] 指定虚拟关键代码进行翻译。
            int uScanCode, // [in] 指定的硬件扫描码的关键须翻译成英文。高阶位的这个值设定的关键，如果是（不压）
            byte[] lpbKeyState, // [in] 指针，以256字节数组，包含当前键盘的状态。每个元素（字节）的数组包含状态的一个关键。如果高阶位的字节是一套，关键是下跌（按下）。在低比特，如果设置表明，关键是对切换。在此功能，只有肘位的CAPS LOCK键是相关的。在切换状态的NUM个锁和滚动锁定键被忽略。
            byte[] lpwTransKey, // [out] 指针的缓冲区收到翻译字符或字符。
            int fuState); // [in] Specifies whether a menu is active. This parameter must be 1 if a menu is active, or 0 otherwise.

        //获取按键的状态
        [DllImport("user32")]
        public static extern int GetKeyboardState(byte[] pbKeyState);


        [DllImport("user32.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.StdCall)]
        private static extern short GetKeyState(int vKey);

        [DllImport("user32.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.StdCall)]
        private static extern short GetAsyncKeyState(int vKey);

        [DllImport("user32.dll")]
        internal static extern uint SendInput(uint nInputs,
            [MarshalAs(UnmanagedType.LPArray), In] INPUT[] pInputs,
            int cbSize);

        #endregion


        public event EventHandler<Keys> KeyUpEvent;
        public event Action<int> OnSpaced;
        public event Action OnBacked;
        public event Action<int> OnPaged;

        public void Start()
        {
            // 安装键盘钩子
            if (hKeyboardHook == 0)
            {
                KeyboardHookProcedure = new HookProc(KeyboardHookProc);

                hKeyboardHook = SetWindowsHookEx(WH_KEYBOARD_LL, KeyboardHookProcedure, GetModuleHandle(Process.GetCurrentProcess().MainModule.ModuleName), 0);

                //如果SetWindowsHookEx失败
                if (hKeyboardHook == 0)
                {
                    Stop();
                    throw new Exception("安装键盘钩子失败");
                }
            }
        }
        public void Stop()
        {
            bool retKeyboard = true;


            if (hKeyboardHook != 0)
            {
                retKeyboard = UnhookWindowsHookEx(hKeyboardHook);
                hKeyboardHook = 0;
            }

            if (!retKeyboard)
            {
                throw new Exception("卸载钩子失败！");
            }
        }

        public void Send(string msg)
        {
            if (!string.IsNullOrEmpty(msg))
            {
                Stop();
                SendKeys.SendWait(msg);
                Start();
            }
        }

        bool isLocked = true;

        public bool IsStarted { set; get; }

        /// <summary>
        /// 按键处理
        /// </summary>
        /// <param name="nCode"></param>
        /// <param name="wParam"></param>
        /// <param name="lParam"></param>
        /// <returns>如果返回1，则结束消息，这个消息到此为止，不再传递。如果返回0或调用CallNextHookEx函数则消息出了这个钩子继续往下传递，也就是传给消息真正的接受者</returns>
        private int KeyboardHookProc(int nCode, Int32 wParam, IntPtr lParam)
        {
            // 侦听键盘事件
            if (nCode >= 0 && wParam == 0x0100)
            {
                KeyboardHookStruct myKeyboardHookStruct = (KeyboardHookStruct) Marshal.PtrToStructure(lParam, typeof(KeyboardHookStruct));

                if (myKeyboardHookStruct.vkCode == 27)
                {
                    Environment.Exit(1);
                }

                #region
                if (isLocked)
                {
                    #region page
                    if (myKeyboardHookStruct.vkCode == 33)
                    {
                        OnPaged?.Invoke(-1);
                    }
                    if (myKeyboardHookStruct.vkCode == 34)
                    {
                        OnPaged?.Invoke(1);
                    }
                    #endregion

                    if (IsStarted && myKeyboardHookStruct.vkCode >= 48 && myKeyboardHookStruct.vkCode <= 57)
                    {
                        var c = int.Parse(((char) myKeyboardHookStruct.vkCode).ToString());
                        OnSpaced?.Invoke(c);
                        IsStarted = false;
                        return 1;
                    }
                    if (IsStarted && myKeyboardHookStruct.vkCode == 8)
                    {
                        OnBacked?.Invoke();

                        return IsStarted ? 1 : 0;
                    }
                    if ((myKeyboardHookStruct.vkCode >= 65 && myKeyboardHookStruct.vkCode <= 90) || myKeyboardHookStruct.vkCode == 32)
                    {
                        if (myKeyboardHookStruct.vkCode >= 65 && myKeyboardHookStruct.vkCode <= 90)
                        {
                            Keys keyData = (Keys) myKeyboardHookStruct.vkCode;
                            KeyUpEvent?.Invoke(this, keyData);
                            IsStarted = true;
                        }
                        if (myKeyboardHookStruct.vkCode == 32)
                        {
                            IsStarted = true;
                            OnSpaced?.Invoke(0);
                        }
                        return IsStarted ? 1 : 0;
                    }
                    else
                    {
                        return 0;
                    }
                }
                #endregion
            }
            return CallNextHookEx(hKeyboardHook, nCode, wParam, lParam);
        }
    }

```

参见：[C# 输入法](http://www.cnblogs.com/yswenli/p/6528447.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。