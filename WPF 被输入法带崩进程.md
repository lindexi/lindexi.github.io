# WPF 被输入法带崩进程

本文记录搜狗输入法某个版本在向 WPF 应用程序输入的时候，让 WPF 应用程序进程退出的问题

<!--more-->

<!-- 发布 -->

我在自定义的文本库里面用了如下代码

```csharp
        [ComImport, Guid("aa80e801-2021-11d2-93e0-0060b067b86e"), InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
        internal interface ITfThreadMgr
        {
            void Activate(out int clientId);
            void Deactivate();
            void CreateDocumentMgr(out IntPtr docMgr);
            void EnumDocumentMgrs(out IntPtr enumDocMgrs);
            void GetFocus(out IntPtr docMgr);
            void SetFocus(IntPtr docMgr);
            void AssociateFocus(IntPtr hwnd, IntPtr newDocMgr, out IntPtr prevDocMgr);
            void IsThreadFocus([MarshalAs(UnmanagedType.Bool)] out bool isFocus);
            void GetFunctionProvider(ref Guid classId, out IntPtr funcProvider);
            void EnumFunctionProviders(out IntPtr enumProviders);
            void GetGlobalCompartment(out IntPtr compartmentMgr);
        }
```

在调用 SetFocus 方法时，也许此时进程就会退出

```csharp
            _hwndSource = (HwndSource) PresentationSource.FromVisual(foo);
            if (_hwndSource == null)
                return;

            //尽管文档说传递null是无效的，但这似乎有助于在与WPF共享的默认输入上下文中激活IME
            var threadMgr = ImeNative.GetTextFrameworkThreadManager();
            threadMgr?.SetFocus(IntPtr.Zero);
```

以上的 GetTextFrameworkThreadManager 方法逻辑如下

```csharp
        internal static ITfThreadMgr GetTextFrameworkThreadManager()
        {
            TF_CreateThreadMgr(out var textFrameworkThreadMgr);
            return textFrameworkThreadMgr;
        }

        [DllImport("msctf.dll")]
        internal static extern int TF_CreateThreadMgr(out ITfThreadMgr threadMgr);
```

调用 SetFocus 方法将会抛出接不住的 AccessViolationException 异常

```
应用程序: Doubi.exe
Framework 版本: v4.0.30319
说明: 由于未经处理的异常，进程终止。
异常信息: System.AccessViolationException
   在 Lindexi.TextEditor.Editing.ImeNative+ITfThreadMgr.SetFocus(IntPtr)
   在 Lindexi.TextEditor.Editing.ImeSupport.CreateContext()
   在 Lindexi.TextEditor.Editing.ImeSupport.<.ctor>b__7_0(System.Object, System.Windows.Input.KeyboardFocusChangedEventArgs)
   在 System.Windows.Input.KeyboardFocusChangedEventArgs.InvokeEventHandler(System.Delegate, System.Object)
   在 System.Windows.RoutedEventArgs.InvokeHandler(System.Delegate, System.Object)
   在 System.Windows.RoutedEventHandlerInfo.InvokeHandler(System.Object, System.Windows.RoutedEventArgs)
   在 System.Windows.EventRoute.InvokeHandlersImpl(System.Object, System.Windows.RoutedEventArgs, Boolean)
   在 System.Windows.UIElement.RaiseEventImpl(System.Windows.DependencyObject, System.Windows.RoutedEventArgs)
   在 System.Windows.UIElement.RaiseTrustedEvent(System.Windows.RoutedEventArgs)
   在 System.Windows.UIElement.RaiseEvent(System.Windows.RoutedEventArgs, Boolean)
   在 System.Windows.Input.InputManager.ProcessStagingArea()
   在 System.Windows.Input.InputManager.ProcessInput(System.Windows.Input.InputEventArgs)
   在 System.Windows.Input.KeyboardDevice.ChangeFocus(System.Windows.DependencyObject, Int32)
   在 System.Windows.Input.KeyboardDevice.TryChangeFocus(System.Windows.DependencyObject, System.Windows.Input.IKeyboardInputProvider, Boolean, Boolean, Boolean)
   在 System.Windows.Input.KeyboardDevice.Focus(System.Windows.DependencyObject, Boolean, Boolean, Boolean)
   在 System.Windows.Input.KeyboardDevice.Focus(System.Windows.IInputElement)
   在 System.Windows.UIElement.Focus()
   在 Lindexi.TextEditor.TextEditor.EnterEdit()
   在 Lindexi.TextEditor.TextEditor.ChangeIsEditableInner(Boolean)
   在 Lindexi.TextEditor.TextEditor.OnIsEditableChanged(System.Windows.DependencyObject, System.Windows.DependencyPropertyChangedEventArgs)
   在 System.Windows.DependencyObject.OnPropertyChanged(System.Windows.DependencyPropertyChangedEventArgs)
   在 System.Windows.FrameworkElement.OnPropertyChanged(System.Windows.DependencyPropertyChangedEventArgs)
   在 Lindexi.TextEditor.TextEditor.OnPropertyChanged(System.Windows.DependencyPropertyChangedEventArgs)
   在 System.Windows.DependencyObject.NotifyPropertyChange(System.Windows.DependencyPropertyChangedEventArgs)
   在 System.Windows.DependencyObject.UpdateEffectiveValue(System.Windows.EntryIndex, System.Windows.DependencyProperty, System.Windows.PropertyMetadata, System.Windows.EffectiveValueEntry, System.Windows.EffectiveValueEntry ByRef, Boolean, Boolean, System.Windows.OperationType)
   在 System.Windows.DependencyObject.SetValueCommon(System.Windows.DependencyProperty, System.Object, System.Windows.PropertyMetadata, Boolean, Boolean, System.Windows.OperationType, Boolean)
   在 Lindexi.Doubi.Extensions.NodeContainer.EditText()
   在 Lindexi.Doubi.Extensions.SubjectNode.EditText()
   在 Lindexi.Doubi.Extensions.DoubiDevice.NodeClick(System.Windows.Input.InputEventArgs, System.Windows.Point)
   在 Lindexi.Doubi.Extensions.DoubiDevice.HandleDown(System.Windows.Input.InputEventArgs, System.Windows.Point)
   在 Lindexi.Doubi.Extensions.DoubiDevice.Node_OnMouseDown(System.Object, System.Windows.Input.MouseButtonEventArgs)
   在 System.Windows.Input.MouseButtonEventArgs.InvokeEventHandler(System.Delegate, System.Object)
   在 System.Windows.RoutedEventArgs.InvokeHandler(System.Delegate, System.Object)
   在 System.Windows.RoutedEventHandlerInfo.InvokeHandler(System.Object, System.Windows.RoutedEventArgs)
   在 System.Windows.EventRoute.InvokeHandlersImpl(System.Object, System.Windows.RoutedEventArgs, Boolean)
   在 System.Windows.UIElement.ReRaiseEventAs(System.Windows.DependencyObject, System.Windows.RoutedEventArgs, System.Windows.RoutedEvent)
   在 System.Windows.UIElement.OnMouseDownThunk(System.Object, System.Windows.Input.MouseButtonEventArgs)
   在 System.Windows.Input.MouseButtonEventArgs.InvokeEventHandler(System.Delegate, System.Object)
   在 System.Windows.RoutedEventArgs.InvokeHandler(System.Delegate, System.Object)
   在 System.Windows.RoutedEventHandlerInfo.InvokeHandler(System.Object, System.Windows.RoutedEventArgs)
   在 System.Windows.EventRoute.InvokeHandlersImpl(System.Object, System.Windows.RoutedEventArgs, Boolean)
   在 System.Windows.UIElement.RaiseEventImpl(System.Windows.DependencyObject, System.Windows.RoutedEventArgs)
   在 System.Windows.UIElement.RaiseTrustedEvent(System.Windows.RoutedEventArgs)
   在 System.Windows.UIElement.RaiseEvent(System.Windows.RoutedEventArgs, Boolean)
   在 System.Windows.Input.InputManager.ProcessStagingArea()
   在 System.Windows.Input.InputManager.ProcessInput(System.Windows.Input.InputEventArgs)
   在 System.Windows.Input.InputProviderSite.ReportInput(System.Windows.Input.InputReport)
   在 System.Windows.Interop.HwndMouseInputProvider.ReportInput(IntPtr, System.Windows.Input.InputMode, Int32, System.Windows.Input.RawMouseActions, Int32, Int32, Int32)
   在 System.Windows.Interop.HwndMouseInputProvider.FilterMessage(IntPtr, MS.Internal.Interop.WindowMessage, IntPtr, IntPtr, Boolean ByRef)
   在 System.Windows.Interop.HwndSource.InputFilterMessage(IntPtr, Int32, IntPtr, IntPtr, Boolean ByRef)
   在 MS.Win32.HwndWrapper.WndProc(IntPtr, Int32, IntPtr, IntPtr, Boolean ByRef)
   在 MS.Win32.HwndSubclass.DispatcherCallbackOperation(System.Object)
   在 System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate, System.Object, Int32)
   在 System.Windows.Threading.ExceptionWrapper.TryCatchWhen(System.Object, System.Delegate, System.Object, Int32, System.Delegate)
   在 System.Windows.Threading.Dispatcher.LegacyInvokeImpl(System.Windows.Threading.DispatcherPriority, System.TimeSpan, System.Delegate, System.Object, Int32)
   在 MS.Win32.HwndSubclass.SubclassWndProc(IntPtr, Int32, IntPtr, IntPtr)
   在 MS.Win32.UnsafeNativeMethods.DispatchMessage(System.Windows.Interop.MSG ByRef)
   在 System.Windows.Threading.Dispatcher.PushFrameImpl(System.Windows.Threading.DispatcherFrame)
   在 System.Windows.Threading.Dispatcher.PushFrame(System.Windows.Threading.DispatcherFrame)
   在 System.Windows.Application.RunDispatcher(System.Object)
   在 System.Windows.Application.RunInternal(System.Windows.Window)
   在 System.Windows.Application.Run(System.Windows.Window)
   在 Lindexi.Doubi.Shell.Program.Main(System.String[])
   在 Lindexi.Doubi.Program.Main(System.String[])
```

暂时不知道有什么解决方法，还请大佬们教教我如何解决

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
