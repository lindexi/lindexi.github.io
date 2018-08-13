
# WPF 开发

本文：我遇到的WPF的坑

<!--more-->



<div id="toc"></div>

## 单例应用在多实例用户无法使用

如果使用NamedPipeServerStream、`Mutex`做单实例，需要传入字符串，这时如果传入一个固定的字符串，会在多用户的时候无法使用。

因为如果在一个用户启动的软件，那么就注册了这个字符串，在另一个用户就无法启动。解决方法是传入`Environment.UserName`。

在构造函数传入`Environment.UserName`有关的字符串就可以在一个用户进行单例，其他用户打开是自己的软件。

```csharp
public partial class App
{
    #region Constants and Fields

    /// <summary>The event mutex name.</summary>
    private const string UniqueEventName = "{GUID}";

    /// <summary>The unique mutex name.</summary>
    private const string UniqueMutexName = "{GUID}"; //这里需要加 Environment.UserName

    /// <summary>The event wait handle.</summary>
    private EventWaitHandle eventWaitHandle;

    /// <summary>The mutex.</summary>
    private Mutex mutex;

    #endregion

    #region Methods

    /// <summary>The app on startup.</summary>
    /// <param name="sender">The sender.</param>
    /// <param name="e">The e.</param>
    private void AppOnStartup(object sender, StartupEventArgs e)
    {
        bool isOwned;
        this.mutex = new Mutex(true, UniqueMutexName, out isOwned);
        this.eventWaitHandle = new EventWaitHandle(false, EventResetMode.AutoReset, UniqueEventName);

        // So, R# would not give a warning that this variable is not used.
        GC.KeepAlive(this.mutex);

        if (isOwned)
        {
            // Spawn a thread which will be waiting for our event
            var thread = new Thread(
                () =>
                {
                    while (this.eventWaitHandle.WaitOne())
                    {
                        Current.Dispatcher.BeginInvoke(
                            (Action)(() => ((MainWindow)Current.MainWindow).BringToForeground()));
                    }
                });

            // It is important mark it as background otherwise it will prevent app from exiting.
            thread.IsBackground = true;

            thread.Start();
            return;
        }

        // Notify other instance so it could bring itself to foreground.
        this.eventWaitHandle.Set();

        // Terminate this instance.
        this.Shutdown();
    }

    #endregion
}
```

## 标记方法被使用

使用 UsedImplicitly 特性可以标记一个没有被引用的方法为反射使用，这时就不会被优化删除。

```csharp
public class Foo
{
    [UsedImplicitly]
    public Foo()
    {
        //反射调用
    }

    public Foo(string str)
    {
        //被引用
    }
}
```

## 当鼠标滑过一个被禁用的元素时，让ToolTip 显示

设置`ToolTipService.ShowOnDisabled`为 true

```csharp
<Button ToolTipService.ShowOnDisabled="True">  
```

## 获取当前域用户

在 WPF 找到当前登陆的用户使用下面代码

```csharp
using System.Security.Principal;

// 其他代码

            WindowsIdentity windowsIdentity = WindowsIdentity.GetCurrent();
            string crentUserAd = windowsIdentity.Name;
```

输出 `crentUserAd` 可以看到 `设备\\用户` 的格式

## 注册全局事件

如果需要注册一个类型的全局事件，如拿到 TextBox 的全局输入，那么可以使用下面代码

```csharp
EventManager.RegisterClassHandler(typeof(TextBox), TextBox.KeyDownEvent, new RoutedEventHandler(方法));
```

## 高版本的 WPF 引用低版本类库导致无法启动

如果在一个 .net 4.0 的 WPF 程序引用一个 .net 2.0 的库，那么就会让程序无法运行，解决方法添加`useLegacyV2RuntimeActivationPolicy`

打开 app.config 添加`useLegacyV2RuntimeActivationPolicy="true"`

下面是 app.config 代码

```csharp
<?xml version="1.0" encoding="utf-8"?>
<configuration>
<startup useLegacyV2RuntimeActivationPolicy="true">
  <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0"/>
</startup>
</configuration>
```

参见：[WPF 软件引用其他类库启动无反应问题 - 灰色年华 - CSDN博客](http://blog.csdn.net/barry_hui/article/details/78758405 )

## 非托管使用托管委托

如果有一个 C++ 写的dll，他需要一个函数指针，在C#使用，就可以传入委托。

那么简单的方法是这样写：


```csharp
    private static void Func(){}
    public void C()
    {
        c(Func);
    }
```

其中c就是C++写的函数，传进去看起来好像正常。

但是有时候程序不知道怎么就炸了。

因为这样写是不对的。

传入的不是函数地址，传入的是把函数隐式转换委托，然后转换的委托是局部变量，会被gc，所以在C++拿到的是一个被回收的委托，调用时就会炸。

这里无法用catch，所以用这个会让程序退出。

调用C#的函数，使用委托，是隐式转换，上面代码可以写成下面的


```csharp
    private static void Func(){}
    public void C()
    {
         var temp=new delegate(){Func};
         c(temp);
    }

```

于是在函数完就把temp放到gc在调用时找不到委托。

一个好的做法


```csharp
    private static void Func(){}
    private delegate Temp=new delegate(){Func};
    private void C()
    {
        c(Temp);
    }
```

放在静态变量不会gc调用不会空，可以这样不会出现上面问题。



## 元素失去获得

元素可以使用 CaptureMouse 方法获得，这可以用在拖动，一旦拖动出元素可以获得，得到拖动结束。

但是有时会失去获得，如果自己需要失去，可以使用 Mouse.Capture(null) 但是在没有自己使用的这个函数，失去获得，可以的是：

设置元素可命中false，如果看到元素失去交互，而且堆栈没有任何地方使用失去获得，那么可能就是存在设置元素可命中false。

如果有两个函数同时 获得 一个元素，会不会出现 失去获得？不会，如果同一个元素多次 获得，那么不会出现失去获得。如果这是让另一个获得，那么这个元素就是失去获得。可以通过元素.IsMouseCaptured 判断元素获得。

可以通过 Mouse.Captured 获得现在 Mouse 是否获得。如果返回是 null ，没有获得，但是元素获得存在一些问题，在失去焦点或其他，可能就失去获得。

[CaptureMouse/CaptureStylus 可能会失败 - walterlv](https://walterlv.github.io/post/wpf/capture-mouse-failed.html )

## 反射引用程序集

这是比较难以说明的问题，总之，可能出现的问题就是引用了一个 xaml 使用的资源库，或使用了一个只有反射才访问的库。

原因：
如果在引用一个库，引用代码没有直接使用的程序集。使用的方法就是使用 xaml 或反射来使用。那么在生成，vs 不会把程序集放在输出文件夹。

问题：
反射报错，无法找到程序集。

例子：
如果我用了一个程序集，然而代码没有直接引用，而是反射使用，这样，vs判断这个程序集没有使用，最后把他清除。所以会出现反射无法拿到，而且很难知道这里出现坑。


为了解决 xaml 和反射无法拿到的坑，可以使用 在任意位置使用 Debug.Write(typeof(程序集里的一个类)) 方法让 vs 引用程序集。


那么在 Release 上为何还可以把程序集放在输出文件夹呢？因为我也不知道原因，如果你知道的话，那么请告诉我一下。

## 使用十进制设置颜色

在 xaml 如果需要使用 十进制设置颜色，请使用下面代码

```csharp
    <SolidColorBrush x:Key="LikeGreen">
        <SolidColorBrush.Color>
            <Color R="100" G="200" B="30" A="100"/>
        </SolidColorBrush.Color>
    </SolidColorBrush>
```

[https://stackoverflow.com/a/47952098/6116637](https://stackoverflow.com/a/47952098/6116637 )

## WPF 判断文件是否隐藏

可以设置一些文件是隐藏文件，那么 WPF 如何判断 FileInfo 是隐藏文件？

简单的代码，通过判断 Attributes 就可以得到，请看下面。


```csharp
    file.Attributes.HasFlag(FileAttributes.Hidden)
```

## 触发鼠标事件

触发鼠标点下事件，可以使用下面代码

```csharp
element.RaiseEvent(new MouseEventArgs(Mouse.PrimaryDevice, 1)
            {
                RoutedEvent = Mouse.MouseDownEvent
            });
```

## TextBlock 换行

使用 `&#10;`

## 在 xaml 绑定索引空格

如果一个索引需要传入空格，那么在 xaml 使用下面代码是无法绑定

```csharp
{Binding MyCollection[foo bar]}
```

需要使用下面代码

```csharp
{Binding MyCollection[[foo&x20;bar]]}
```

[Binding to an index with space in XAML – Ivan Krivyakov](https://ikriv.com/blog/?p=1143 )


[wpf动画——new PropertyPath属性链 - 影天 - 博客园](http://www.cnblogs.com/xwlyun/archive/2012/09/14/2685199.html)

[wpf动画——缓动动画Animation Easing - 影天 - 博客园](http://www.cnblogs.com/xwlyun/archive/2012/09/11/2680579.html)






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。