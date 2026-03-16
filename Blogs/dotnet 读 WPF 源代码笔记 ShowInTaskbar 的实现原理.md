---
title: dotnet 读 WPF 源代码笔记 ShowInTaskbar 的实现原理
description: 本文记录我阅读 WPF 项目了解到的 ShowInTaskbar 的实现原理
tags: WPF,dotnet
category: 
---

<!-- CreateTime:2026/03/10 07:15:25 -->

<!-- 发布 -->
<!-- 博客 -->

在 WPF 里面，通过判断 `ShowInTaskbar == false` 决定是否在任务栏隐藏窗口图标，其实现原理是创建一个隐藏的窗口，将当前窗口的 Owner 设置为此隐藏窗口而实现

在 `Window.EnsureHiddenWindow` 方法里面将创建名为 `Hidden Window` 的隐藏窗口，这个隐藏窗口的唯一作用就是在 `ShowInTaskbar == false`  时，被设置为当前窗口的 Owner 从而实现隐藏窗口的功能

```csharp
        /// <summary>
        ///     Get or create the hidden window used for parenting when ShowInTaskbar == false.
        /// </summary>
        private HwndWrapper EnsureHiddenWindow()
        {
            if (_hiddenWindow == null)
            {
                _hiddenWindow = new HwndWrapper(
                    0, // classStyle
                    NativeMethods.WS_OVERLAPPEDWINDOW, // style
                    0, // exStyle
                    NativeMethods.CW_USEDEFAULT, // x
                    NativeMethods.CW_USEDEFAULT, // y
                    NativeMethods.CW_USEDEFAULT, // width
                    NativeMethods.CW_USEDEFAULT, // height
                    "Hidden Window", // name
                    IntPtr.Zero,
                    null
                );
            }

            return _hiddenWindow;
        }
```

在 ShowInTaskbar 变更的时候，将先调用 EnsureHiddenWindow 方法让 `_hiddenWindow` 完成执行初始化，而后再调用 `SetOwnerHandle(_hiddenWindow.Handle);` 将隐藏窗口为当前的窗口的 Owner 从而实现在任务栏隐藏窗口图标的功能

```csharp
        /// <summary>
        ///     The DependencyProperty for ShowInTaskbarProperty.
        ///     Flags:              None
        ///     Default Value:      true
        /// </summary>
        public static readonly DependencyProperty ShowInTaskbarProperty =
                DependencyProperty.Register("ShowInTaskbar",
                        typeof(bool),
                        typeof(Window),
                        new FrameworkPropertyMetadata(BooleanBoxes.TrueBox,
                                new PropertyChangedCallback(_OnShowInTaskbarChanged),
                                new CoerceValueCallback(VerifyAccessCoercion)));

        private static void _OnShowInTaskbarChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {
            Window w = (Window)d;

            Debug.Assert(w != null, "DependencyObject must be of type Window.");
            w.OnShowInTaskbarChanged();
        }

        private void OnShowInTaskbarChanged()
        {
            ...
                    SetTaskbarStatus();
            ...
        }

        /// <summary>
        ///     sets taskbar status
        /// </summary>
        private void SetTaskbarStatus()
        {
            if (ShowInTaskbar == false) // don't show in taskbar
            {
                // To remove the taskbar button for this window it needs to have a non-null parent
                // (we'll create a hidden window for this purpose) and not have WS_EX_APPWINDOW

                // Create this now, even if we're not currently going to parent it.
                // If the Owner changes, we'll need to switch to this.
                EnsureHiddenWindow();

                // when this window is unowned
                if (_ownerHandle == IntPtr.Zero)
                {
                    SetOwnerHandle(_hiddenWindow.Handle);

                    ...
                }

                ...
            }
            else // (ShowInTaskbar == true) show in task bar
            {
                ...
            }
        }
```

更多关于 ShowInTaskbar 请参阅：

[WPF 设置 ShowInTaskbar 对窗口最小化的影响](https://blog.lindexi.com/post/WPF-%E8%AE%BE%E7%BD%AE-ShowInTaskbar-%E5%AF%B9%E7%AA%97%E5%8F%A3%E6%9C%80%E5%B0%8F%E5%8C%96%E7%9A%84%E5%BD%B1%E5%93%8D.html )

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建
