# WPF shows that some windows in multithreading will be locked in the PenThreadWorker constructor when the application starts

**Problem description:**

WPF will initialize pen thread when the window initializing. But I find the window may be locked in PenThreadWorker constructor.

The `UnsafeNativeMethods.CreateResetEvent` will lock the thread.
 
 **Actual behavior:** <!-- callstack for crashes / exceptions -->

I use multithreading to create some windows by this code and maybe the thread be locked in PenThreadWorker constructor.

The Demo code

```csharp
    public partial class App : Application
    {
        /// <inheritdoc />
        protected override void OnStartup(StartupEventArgs e)
        {
            var thread = new Thread(() =>
            {
                var mainWindow = new MainWindow();
                mainWindow.Show();
                Dispatcher.Run();
            });
            thread.SetApartmentState(ApartmentState.STA);
            thread.Start();

            base.OnStartup(e);
        }
    }
```

The step

1. Run the code 

1. Use the VisualStudio to suspend the application

Maybe you can find the application only create a `Stylus Input` thread and show only a main window and the other main window and the thread be locked in PenThreadWorker constructor and the window can not show

Why I need multithreading to show the windows? I need the splash window to show the welcome page and then I should show the main window in other thread to do the bussiness code and I should close the splash window when the main window showed.

But I found some users could not show the main window and the main window be lock in PenThreadWorker constructor.

I use dnspy to suspend the application and find the code run in PenThreadWorker constructor and the `UnsafeNativeMethods.CreateResetEvent` will lock the thread.

```csharp
 PenThreadWorker..ctor()  
 PenThread..ctor()  
 PenThreadPool.GetPenThreadForPenContextHelper(PenContext penContext)  
 PenThreadPool.GetPenThreadForPenContext(PenContext penContext)  
 WispTabletDeviceCollection.UpdateTabletsImpl()  
 WispTabletDeviceCollection.UpdateTablets()  
 WispTabletDeviceCollection..ctor()  
 WispLogic.get_WispTabletDevices()  
 WispLogic.RegisterHwndForInput(InputManager inputManager, PresentationSource inputSource)  
 HwndStylusInputProvider..ctor(HwndSource source)  
 HwndSource.Initialize(HwndSourceParameters parameters)  
 HwndSource..ctor(HwndSourceParameters parameters)  

 PresentationFramework.dll!System.Windows.Window.CreateSourceWindow(bool duringShow)  
 PresentationFramework.dll!System.Windows.Window.CreateSourceWindowDuringShow()  
 PresentationFramework.dll!System.Windows.Window.ShowHelper(object booleanBox)  
```



 **Expected behavior:**

The `UnsafeNativeMethods.CreateResetEvent` do not lock the thread
 
 **Minimal repro:**
 
Create a empty WPF application and then change the app.xaml.cs code

```csharp
    public partial class App : Application
    {
        /// <inheritdoc />
        protected override void OnStartup(StartupEventArgs e)
        {
            var thread = new Thread(() =>
            {
                var mainWindow = new MainWindow();
                mainWindow.Show();
                Dispatcher.Run();
            });
            thread.SetApartmentState(ApartmentState.STA);
            thread.Start();

            base.OnStartup(e);
        }
    }
```

Run the code and maybe you can find one of the main window can not show and it stop in `UnsafeNativeMethods.CreateResetEvent` and I wait for a hour but it can not run.
