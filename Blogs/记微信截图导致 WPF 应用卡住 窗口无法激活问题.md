本文记录一位用户和我反馈的微信截图开启之后 WPF 应用就会卡住的问题，此时的行为就是任何程序的窗口都无法被激活，只有从任务管理器干掉 WPF 应用的进程才能恢复

<!--more-->


<!-- CreateTime:2024/04/18 07:01:58 -->

<!-- 发布 -->
<!-- 博客 -->

我拿了 WPF 应用的堆栈信息如下，看起来是卡在 UXTheme.dll 上，但是不知道里面在执行什么逻辑

```
> user32.dll!_NtUserMessageCall@28() 
  user32.dll!_RealDefWindowProcW@16()
  uxtheme.dll!DoMsgDefault(struct _THEME_MSG const *)
  uxtheme.dll!_ThemeDefWindowProc(struct HWND__ *,unsigned int,unsigned int,long,int)
  uxtheme.dll!_ThemeDefWindowProcW@16()
  user32.dll!_DefWindowProcW@16()
  user32.dll!_InternalCallWinProc@20() 
  user32.dll!_UserCallWinProcCheckWow@32() 
  user32.dll!_CallWindowProcAorW@24()
  user32.dll!_CallWindowProcW@20() 
  [托管到本机的转换]  
  WindowsBase.dll!MS.Win32.HwndSubclass.DefWndProcWrapper(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam) 
  [本机到托管的转换]  
  user32.dll!_InternalCallWinProc@20() 
  user32.dll!_UserCallWinProcCheckWow@32() 
  user32.dll!_CallWindowProcAorW@24()
  user32.dll!_CallWindowProcW@20() 
  [托管到本机的转换]  
  WindowsBase.dll!MS.Win32.HwndSubclass.SubclassWndProc(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam) 
  [本机到托管的转换]  
  user32.dll!_InternalCallWinProc@20() 
  user32.dll!_UserCallWinProcCheckWow@32() 
  user32.dll!_DispatchClientMessage@24() 
  user32.dll!___fnDWORD@4()
  ntdll.dll!7728013a() 
  [下面的框架可能不正确和/或缺失，没有为 ntdll.dll 加载符号]  
  [托管到本机的转换]  
  WindowsBase.dll!System.Windows.Threading.Dispatcher.GetMessage(ref System.Windows.Interop.MSG msg, System.IntPtr hwnd, int minMessage, int maxMessage) 
  WindowsBase.dll!System.Windows.Threading.Dispatcher.PushFrameImpl(System.Windows.Threading.DispatcherFrame frame = {System.Windows.Threading.DispatcherFrame}) 
  WindowsBase.dll!System.Windows.Threading.Dispatcher.PushFrame(System.Windows.Threading.DispatcherFrame frame = {System.Windows.Threading.DispatcherFrame}) 
  WindowsBase.dll!System.Windows.Threading.Dispatcher.Run()
  PresentationFramework.dll!System.Windows.Application.RunDispatcher(object ignore = null) 
  PresentationFramework.dll!System.Windows.Application.RunInternal(System.Windows.Window window = null)
  PresentationFramework.dll!System.Windows.Application.Run(System.Windows.Window window = null)
  PresentationFramework.dll!System.Windows.Application.Run() 
  lindexi.dll!lindexi.Program.Main(string[] args = {string[2]}) 
  [本机到托管的转换]  
  hostpolicy.dll!542463c4()
  hostpolicy.dll!542465c2()
  hostpolicy.dll!54246d75()
  hostfxr.dll!72bf9ad3() 
  hostfxr.dll!72bfb909() 
  hostfxr.dll!72bfd594() 
  hostfxr.dll!72bfbc98() 
  hostfxr.dll!72bf6f1b() 
  lindexi.exe!003701ed()
  lindexi.exe!0037053a()
  lindexi.exe!0037166c()
  kernel32.dll!765a344d()
  ntdll.dll!772a9802() 
  ntdll.dll!772a97d5() 
```

换成 QQ 截图没有能够复现问题，期待后续腾讯的伙伴能够帮忙修复
