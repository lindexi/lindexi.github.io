# WPF will break when an exception be throw in the StylusPlugIn

<!--more-->
<!-- CreateTime:2019/10/7 12:21:19 -->

<!-- csdn -->

We can write a class that inherits the StylusPlugIn. And this class can get the touch event fast in stylus input thread in the overwrite method, such as `OnStylusDown` and the OnStylusUp method. The *Stylus Input* thread is backgroud thread. As we all know, any exception thrown in a background thread will destroy the application. We are hard to catch the background thread and recover this thread. Though we can use `AppDomain.CurrentDomain.UnhandledException` and add `<legacyUnhandledExceptionPolicy enabled="1"/> ` to app.config to catch it. 

The `StylusPlugIn` can be inherited and we can overwrite the method. And we can not stop our friend add his code in the method, such as OnStylusDown method. But we may not enough careful in our code that the code will throw the exception in unexpected in the OnStylusDown method. And the exception will break the stylus input thread and the WPF application will stop responding the touch. And we can not do something to recover the touch responding. Just like this code. The code will break the Stylus Input thread.

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        StylusPlugIns.Add(new Foo());
    }
}

public class Foo : StylusPlugIn
{
    /// <inheritdoc />
    protected override void OnStylusDown(RawStylusInput rawStylusInput)
    {
        throw new Exception();
    }
}
```

The OnStylusDown running in Stylus Input thread and if some friend throws any exceptions that will break the thread. And the application will stop responding touch. 

Of course, the actual situation may be more complicated. Maybe some of my friends just didn't realize that the code he wrote was unstable. But add `try catch` to all the method is an evil code.

But I do not think we can throw any unintended exceptions in StylusPlugIn.

See [StylusPlugIn Class (System.Windows.Input.StylusPlugIns)](https://docs.microsoft.com/en-us/dotnet/api/system.windows.input.stylusplugins.stylusplugin?view=netframework-4.8 )

> If you use a StylusPlugIn inside a control, you should test the plug-in and control extensively to make sure they do not throw any unintended exceptions.

See [Intercepting Input from the Stylus](https://msdn.microsoft.com/en-us/data/ms749105(v=vs.80) )

> If a StylusPlugIn throws or causes an exception, the application will close. You should thoroughly test controls that consume a StylusPlugIn and only use a control if you are certain the StylusPlugIn will not throw an exception.

Just as the document say, we should make sure the code do not throw any unintended exceptions in StylusPlugIn. But actual we are hard to do it in the complex business.

I think it may be a good way that we can catch all the exceptions in the code that fire the StylusPlugIn method.

We are making a full-screen touch application and the user can only control the machine by touch input. If the WPF application stops responding touch, the user can not do something except reboot. 

All the demo code in [github](https://github.com/lindexi/lindexi_gd/tree/4f1cda37f1a6eb4fc88fa404b104cbf9b29b365e/KihemjaibeaNafebahearjece)

[WPF will break when an exception be throw in the StylusPlugIn · Issue #1037 · dotnet/wpf](https://github.com/dotnet/wpf/issues/1037 )