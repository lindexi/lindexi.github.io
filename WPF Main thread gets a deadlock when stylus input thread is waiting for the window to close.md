# WPF Main thread gets a deadlock when stylus input thread is waiting for the window to close

We found two way that can make the main thread locked. And we can not write any code to solve it and it can only be circumvented.

The easiest way to reproduce this issue is to wait for the window in the main thread to close in the stylus input thread.

<!--more-->
<!-- CreateTime:2018/11/1 9:32:42 -->

<!-- csdn -->

We have found two ways, the first way always happens, and the second way is probabilistic.

Before we tell you about it, we need to tell you something about the touch thread and why it can make the main thread wait forever.

## Theory

The stylus input thread gets the input event when the user touches the screen.

There is a `ThreadProc` method running in the stylus input thread and this method has a loop inside which will never end until the application exists.

```csharp
void ThreadProc()
{
    // The loop that never ends.
    while (!__disposed)
    {

    }
}
```

There are nested two loops in the `ThreadProc` method. In the outside one, it adds and removes `PenContext` and in the inside one, it will be blocked by the `PENIMC` and can be continued when the user touches the screen.

```csharp
void ThreadProc()
{
    while (!__disposed)
    {
    	// The outside loop
    	// To remove or add the PenContext

    	while (true)
    	{
    		// The inside loop
    		// Tt will be blocked by the PENIMC
    		if(!Penimc.UnsafeNativeMethods.GetPenEvent(/*the thread locker*/))
    		{
    			// If the `_pimcResetHandle` is released, this if branch will enter so the inside loop will end with the `break` and the code runs back to the outside loop.
    			break;
    		}

    		FireEvent(/*fire the touch events*/);
    	}
    }
}
```

When a window is closed, it calls `HwndSource.DisposeStylusInputProvider` and this causes the `PenContext.Disable` be calling with the calling stack trace showing below.

```
System.Windows.Input.PenThreadWorker.WorkerRemovePenContext(System.Windows.Input.PenContext penContext) 
System.Windows.Input.PenContext.Disable(bool shutdownWorkerThread) 
System.Windows.Input.PenContexts.Disable(bool shutdownWorkerThread) 
System.Windows.Input.StylusWisp.WispLogic.UnRegisterHwndForInput(System.Windows.Interop.HwndSource hwndSource) 
System.Windows.Interop.HwndStylusInputProvider.Dispose() 
```

Let us see the `PenThreadWorker.WorkerRemovePenContext` that run in the main thread.

```csharp
internal bool WorkerRemovePenContext(PenContext penContext)
{
    var operationRemoveContext = new PenThreadWorker.WorkerOperationRemoveContext(penContext, this);

    _workerOperation.Add((PenThreadWorker.WorkerOperation) operationRemoveContext);
    // Release the _pimcResetHandle lock 
    UnsafeNativeMethods.RaiseResetEvent(this._pimcResetHandle.Value);
    // Wait for the operationRemoveContext to finish
    operationRemoveContext.DoneEvent.WaitOne();
    operationRemoveContext.DoneEvent.Close();
    return operationRemoveContext.Result;
}
```

From the code above we can learn that the main thread releases the `_pimcResetHandle` and it makes the `ThreadProc` breaking the inside loop and go back to the outside one to remove the `PenContext`.

Normally we should run the code in the stylus input thread to remove the `PenContext` and keep the main thread waiting for the `operationRemoveContext` to finish. But if the stylus input thread never remove the `PenContext` and the main thread waits for it, the main thread will never continue.

## The first way

The first way is to write a custom class implementing `StylusPlugIn` and wait for a window to close in the `OnStylusUp` method.

Let's create a new empty window named `FooWindow`.


```csharp
public class FooWindow : Window
{

}
```

Then we create a `FooStylusPlugIn` class to implement the `StylusPlugIn` with overriding the `OnStylusUp` method. We add some code to wait for the window to close by calling `Invoke` which will wait by pumping a new message loop.

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

To combine both the critical codes above, we write some codes in the `MainWindow`. The `FooWindow` is instanced in the constructor and the `StylusPlugIn` is plugged in it. We also make a button in the XAML that can let us know whether the main thread is still running or not.

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

Run the project, touch the main window, and you'll find that the main window never responds to your interaction. Try to click the button to view the responding and you'll soon verify what I'm talking.

The reason is that the `OnStylusUp` in `FooStylusPlugIn` is running in the stylus input thread which is also running the inside loop of the `ThreadProc` method. It needs to go back to the outside loop to remove the `PenContext` when a window is closed. The stylus input thread is waiting for the main thread to close a window and the main thread is also waiting for the stylus input thread remove PenContext. Thus, the deadlock occurred.

The demo in [github](https://github.com/dotnet-campus/wpf-issues/tree/master/MainThreadDeadlockWithStylusInputThread/MainThreadDeadlockWhenTouchThreadWaitForWindowClosed)

## The second way

If a touch happens exactly during a window closing, the main thread will enter a lock.

The difference between the first method and the second method is that the first one will lock both the main thread and the stylus input thread but the second one will only lock the main thread.

From the theory, we know that the `PenContext` should be removed correctly in the outside loop. But in the second way, the stylus input thread is firing the touch event exactly when we run the code to remove the `PenContext` in the stylus input thread. As you can see we need to run the code to remove `PenContext` in the outside loop but at this moment the code is firing the touch event in the second loop.

The firing of the touch event means the `_pimcResetHandle` is released. Although the main thread has also released the lock the code cannot run to the outside loop to remove the `PenContext` and the main thread can no longer wait for the moment when the `PenContext` removal is finished.


```csharp
void ThreadProc()
{
    while (!__disposed)
    {
      	// The outside loop
    	// To remove or add the PenContext
    	// The main thread is waiting for its finishing.
    	RemovePenContext();

    	while (true)
    	{
    		// The inside loop
    		// Tt will be blocked by the PENIMC
    		if(!Penimc.UnsafeNativeMethods.GetPenEvent(/*wait the lock*/))
    		{
    			// If the `_pimcResetHandle` is released, this if branch will enter so the inside loop will end with the `break` and the code runs back to the outside loop.
    			break;
    		}

    		FireEvent(/*fire the touch events*/); // the code is running in this line
    		// and the `_pimcResetHandle` is released.
    		// the main thread release the `_pimcResetHandle` but the code can not go to RemovePenContext for it will no longer break. 
    	}
    }
}
```

The main thread has released the lock but the stylus input thread doesn't need to wait for the lock. The stylus input thread cannot go back to the outside loop to remove the `PenContext` and main thread can no longer wait for the moment when the `PenContext` removal is finished.

Thanks to [walterlv](https://walterlv.com/) for proofreading the English translation of this post.

感谢 [吕毅](https://walterlv.com/) 对本文的英文翻译进行校对。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  