
# WPF Main thread gets a deadlock when stylus input thread is waiting for the window to close

We found two way that can make the main thread locked. And we can not write any code to solve it and it can only be circumvented.
The easiest way is to wait for the window in the main thread to close in the stylus input thread.

<!--more-->


<!-- csdn -->

We have found two ways, the first way always happens, and the second way is probability.

Before we tell you about it, we need to tell you something about the touch thread and why it can make the main thread wait forever.

## Theory

In WPF we need stylus input thread to get the input message from the screen.

In stylus input thread, it will enter the `ThreadProc` that has a loop and the loop will never exit until the application exited.

```csharp
void ThreadProc()
{
    // the loop that never exit
    while (!__disposed)
    {

    }
}
```

There are two layer loop in `TheadProc`. It can add or remove PenContext in the first layer loop. And the second layer loop will be locked by PENIMC until something release the thread lock or the user touch the screen. 

```csharp
void ThreadProc()
{
    while (!__disposed)
    {
    	// The first layer loop
    	// To remove or add PenContext

    	while (true)
    	{
    		// the second layer loop
    		// it will be locked by PENIMC
    		if(!Penimc.UnsafeNativeMethods.GetPenEvent(/*the thread locker*/))
    		{
    			// if the thread lock `_pimcResetHandle` be release, it will enter this branch and break the second layer loop and then goto the first layer loop.
    			break;
    		}

    		FireEvent(/*send the touch message*/);
    	}
    }
}
```

It needs to call `HwndSource.DisposeStylusInputProvider` when the window closed. And the call stack is from the message loop to `PenContext.Disable` and the code is below.

```csharp
 	PresentationCore.dll!System.Windows.Input.PenThreadWorker.WorkerRemovePenContext(System.Windows.Input.PenContext penContext) 
	PresentationCore.dll!System.Windows.Input.PenContext.Disable(bool shutdownWorkerThread) 
 	PresentationCore.dll!System.Windows.Input.PenContexts.Disable(bool shutdownWorkerThread) 
 	PresentationCore.dll!System.Windows.Input.StylusWisp.WispLogic.UnRegisterHwndForInput(System.Windows.Interop.HwndSource hwndSource) 
 	PresentationCore.dll!System.Windows.Interop.HwndStylusInputProvider.Dispose() 
```

Let us see the `PenThreadWorker.WorkerRemovePenContext` that run in main thread.

```csharp
    internal bool WorkerRemovePenContext(PenContext penContext)
    {

      var operationRemoveContext = new PenThreadWorker.WorkerOperationRemoveContext(penContext, this);
     
      _workerOperation.Add((PenThreadWorker.WorkerOperation) operationRemoveContext);
      // release _pimcResetHandle lock 
      UnsafeNativeMethods.RaiseResetEvent(this._pimcResetHandle.Value);
      // waiting the operationRemoveContext finished
      operationRemoveContext.DoneEvent.WaitOne();
      operationRemoveContext.DoneEvent.Close();
      return operationRemoveContext.Result;
    }
```

We can know somthing in the code. The main thread release the `_pimcResetHandle` lock and it can make the ThreadProc break the second layer loop and goto the the first layer loop to remove the PenContext.

And we should run the code to remove PenContext in touch thread and the main thread should wating the operationRemoveContext finished.

The main thread should wait the touch thread finished removing PenContext. If the touch thread never remove PenContext, the main thread will never continue.

The demo program about it is in the [github](https://github.com/dotnet-campus/wpf-issues/tree/master/MainThreadDeadlockWithStylusInputThread/MainThreadDeadlockWhenTouchThreadWaitForWindowClosed).

## The first way

The first way is adding a StylusPlugIn and in OnStylusUp waiting for a window created in the main window to close.

We should add a window class that is an empty class.


```csharp
    public class FooWindow : Window
    {

    }
```

And we should create a class FooStylusPlugIn that inherit StylusPlugIn. The FooStylusPlugIn override the OnStylusUp and waiting for a window close in OnStylusUp.

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

We create FooWindow and FooStylusPlugIn in main windows and we also need to create a button in xaml that can make us know whether the main thread is running.

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

We can run this code and touch the main window and then we will find the button in the main window can not be clicked.

The reason about it is the OnStylusUp in FooStylusPlugIn is running in stylus input thread and it running in the second layer loop in ThreadProc. And it needs to go to the first layer loop to remove the PenContext when a window closed. The touch thread is waiting for the main thread close a window and the main thread is waiting for the touch thread remove PenContext.



## The second way

When touch occured and at the same time the window  closed that will make the main thread be lock.

The difference between the first method and the second method is that the first way will lock the main thread and the touch thread but the second way will only lock the main thread.

We can know from the theory that we should remove the PenContext in the first layer loop. The touch thread is firing the touch message exact when we run the code to remove PenContext in touch thread. As you can see we need to run the code to remove PenContext in the first layer loop but now the code is firing the touch message in the second loop.

If the touch thread is firing the touch message that means the `_pimcResetHandle` lock is released. Although the main thread also release the lock but we can not go to the first layer loop to remove the PenContext and the main thread can not never wait for the PenContext removed finished.


```csharp
void ThreadProc()
{
    while (!__disposed)
    {
      	// The first layer loop
    	// To remove or add PenContext
    	// The main thread is waiting for it finished.
    	RemovePenContext();

    	while (true)
    	{
    		// the second layer loop
    		// it will be locked by PENIMC
    		if(!Penimc.UnsafeNativeMethods.GetPenEvent(/*wait lock*/))
    		{
    			// if the thread lock `_pimcResetHandle` be released, it will enter this branch and break the second layer loop and then goto the first layer loop.
    			break;
    		}

    		FireEvent(/*send the touch message*/); // the code is running in this line
    		// and the `_pimcResetHandle` is released.
    		// the main thread release the `_pimcResetHandle` but the code can not go to RemovePenContext for it can not break. 
    	}
    }
}
```

The main thread release lock but the touch thread need not wait for the lock. The touch thread can not go to the code to remove PenContext and the main thread will never wait for the PenContext removed finished.





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。