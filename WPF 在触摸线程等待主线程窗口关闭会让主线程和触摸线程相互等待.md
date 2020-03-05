# WPF 在触摸线程等待主线程窗口关闭会让主线程和触摸线程相互等待

本文是记录一个线程相互等待导致主线程无法响应的问题，这个问题是属于一定可以复现的问题，是 WPF 的已知问题。如果遇到这个问题，属于暂时没有方法解决，只能规避。

这个问题的最简单复现步骤是在触摸线程，也就是 StylusInput 线程，等待一个主线程的窗口关闭，此时就会出现主线程卡住的问题

<!--more-->
<!-- CreateTime:2018/10/31 9:30:09 -->

<!-- csdn -->

这个问题有两个复现方法，第一个方法属于必现的方法，第二个方法属于概率的方法

在开始说明问题之前需要大概讲一下 WPF 的触摸原理和这个问题的原理

## 原理

在 WPF 触摸下，是存在 Stylus Input 线程用于处理触摸相关的事情，在这个线程会调用 ThreadProc 进入循环

<!-- ![](image/WPF 在触摸线程等待主线程窗口关闭会让主线程和触摸线程相互等待/WPF 在触摸线程等待主线程窗口关闭会让主线程和触摸线程相互等待0.png) -->

这个线程会调用 ThreadProc 进入循环，直到软件退出

```csharp
void ThreadProc()
{

}
```

在 ThreadProc 里面有两次循环，第一层循环是处理添加或移除 PenContext 等，第二层循环是进入 PENIMC 这个库卡住，直到释放线程锁 `_pimcResetHandle` 或用户触摸才继续

```csharp
void ThreadProc()
{
    while (!__disposed)
    {
    	// 第一层循环
    	// 处理 PenContext 的添加或移除等的代码

    	while (true)
    	{
    		// 第二层循环，用于处理用户的触摸
    		if(!Penimc.UnsafeNativeMethods.GetPenEvent(/*等待 _pimcResetHandle 释放，或用户触摸*/))
    		{
    			// 如果是 _pimcResetHandle 被释放，则跳出第二层循环
    			break;
    		}

    		FireEvent(/*触发触摸消息*/);
    	}
    }
}
```

在窗口关闭的时候，需要调用 `HwndSource.DisposeStylusInputProvider` 关闭窗口的触摸，这时的调用堆栈是从消息到 `PenContext.Disable` 方法

```csharp
 	PresentationCore.dll!System.Windows.Input.PenThreadWorker.WorkerRemovePenContext(System.Windows.Input.PenContext penContext) 
	PresentationCore.dll!System.Windows.Input.PenContext.Disable(bool shutdownWorkerThread) 
 	PresentationCore.dll!System.Windows.Input.PenContexts.Disable(bool shutdownWorkerThread) 
 	PresentationCore.dll!System.Windows.Input.StylusWisp.WispLogic.UnRegisterHwndForInput(System.Windows.Interop.HwndSource hwndSource) 
 	PresentationCore.dll!System.Windows.Interop.HwndStylusInputProvider.Dispose() 
```

先来看一下 `PenThreadWorker.WorkerRemovePenContext` 的代码

```csharp
    internal bool WorkerRemovePenContext(PenContext penContext)
    {

      var operationRemoveContext = new PenThreadWorker.WorkerOperationRemoveContext(penContext, this);
     
      _workerOperation.Add((PenThreadWorker.WorkerOperation) operationRemoveContext);
      // 释放 _pimcResetHandle 锁
      UnsafeNativeMethods.RaiseResetEvent(this._pimcResetHandle.Value);
      // 等待任务完成
      operationRemoveContext.DoneEvent.WaitOne();
      operationRemoveContext.DoneEvent.Close();
      return operationRemoveContext.Result;
    }
```

从上面的代码可以看到，主线程需要等待 WorkerOperationRemoveContext 运行完成，而 WorkerOperationRemoveContext 需要在 Stylus Input 线程运行

这就是关闭窗口可能出现的主线程卡住问题，只要主线程等待没有完成，主线程就会一直等待

## 方法一

添加一个 StylusPlugIn 同时在 StylusPlugIn 的 Up 方法等待一个窗口的关闭

在代码添加一个窗口类，这个窗口类是一个空白的窗口

```csharp
    public class FooWindow : Window
    {

    }
```

然后创建一个类 FooStylusPlugIn 继承 StylusPlugIn 类，重写 OnStylusUp 方法，在这个方法等待传入的 FooWindow 关闭

```csharp
   public class FooStylusPlugIn : StylusPlugIn
    {
        public FooStylusPlugIn(FooWindow fooWindow)
        {
            FooWindow = fooWindow;
        }

        public FooWindow FooWindow { get; }

        /// <inheritdoc />
        protected override void OnStylusUp(RawStylusInput rawStylusInput)
        {
            FooWindow.Dispatcher.Invoke(() => FooWindow.Close());
            base.OnStylusUp(rawStylusInput);
        }
    }
```

在主窗口创建 FooWindow 和 FooStylusPlugIn 同时在前台放一个按钮，放一个按钮可以知道当前的主线程是否无法点击

```csharp
   public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            _fooWindow = new FooWindow();
            StylusPlugIns.Add(new FooStylusPlugIn(_fooWindow));
            _fooWindow.Show();
        }

        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
        }

        private FooWindow _fooWindow;
    }
```

这时运行代码触摸一下屏幕就会发现主窗口的按钮无法点击

因为在 FooStylusPlugIn 的 OnStylusUp 属于 Stylus Input 线程，执行的方法在 ThreadProc 的 FireEvent 里，而处理窗口关闭的时候需要调用 WorkerOperationRemoveContext 也需要在 Stylus Input 线程运行。

在主线程需要等待触摸线程运行移除 PenContext 代码，触摸线程需要等待主线程关闭窗口，这时两个线程就无响应

所有的代码在 [github](https://github.com/dotnet-campus/wpf-issues/tree/master/MainThreadDeadlockWithStylusInputThread/MainThreadDeadlockWhenTouchThreadWaitForWindowClosed)

## 方法二

在触摸触发的过程中，出现了窗口的关闭，会让主线程卡住

和方法一不同的是，方法一会让触摸线程和主线程同时卡住，方法二只会让主线程卡住

从原理上可以知道，窗口关闭需要移除 PenContext 需要在触摸线程的第一层循环运行。但是在触摸的过程，触摸线程运行到第二层循环里。

```csharp
void ThreadProc()
{
    while (!__disposed)
    {
    	// 第一层循环
    	// 处理 PenContext 的添加或移除等的代码
    	// 主线程需要等待这里的代码运行完成
    	RemovePenContext();

    	while (true)
    	{
    		// 第二层循环，用于处理用户的触摸
    		if(!Penimc.UnsafeNativeMethods.GetPenEvent(/*等待 _pimcResetHandle 释放，或用户触摸*/))
    		{
    			// 如果是 _pimcResetHandle 被释放，则跳出第二层循环
    			break;
    		}

    		FireEvent(/*触发触摸消息*/); // 当前触摸线程运行到这里
    	}
    }
}
```

在没有出现触摸的时候，触摸线程会在 `Penimc.UnsafeNativeMethods.GetPenEvent` 卡住

主线程通过释放 `_pimcResetHandle` 锁运行 RemovePenContext 代码

触摸线程在运行到 FireEvent 不需要等待`_pimcResetHandle`就无法到第一层循环，主线程无法等到触摸线程移除 PenContext 主线程卡住

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
