# WPF 开发

本文：我遇到的WPF的坑

<!--more-->
<!-- CreateTime:2019/12/27 8:31:20 -->


<div id="toc"></div>


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

## 拼接 URI 路径

我需要将一个 URI 和另一个 URI 拼接如 `https://blog.lindexi.com/post/123` 和 `/api/12` 拼接，拿到绝对路径 `https://blog.lindexi.com/api/12` 可以使用下面方法

```csharp
var uri1 = new Uri("https://blog.lindexi.com/post/123");
var uri2 = "/api/12";

    if (Uri.TryCreate(uri1, uri2, out var absoluteUrl))
    {
        // 拼接成功，在这里就可以使用 absoluteUrl 拼接后的绝对路径
    }
```

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

## 当鼠标滑过一个被禁用的元素时，让ToolTip 显示

设置`ToolTipService.ShowOnDisabled`为 true

```csharp
<Button ToolTipService.ShowOnDisabled="True">  
```

## 获取设备屏幕数量

通过 WinForms 方法获取

```csharp
System.Windows.Forms.Screen.AllScreens
```

上面就可以拿到所有的屏幕，通过 Count 方法就可以知道有多少屏幕

```csharp
var screenCount = Screen.AllScreens.Length;
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

## 绑定资源文件里面的资源

在 WPF 的 xaml 可以通过 `x:Static` 绑定资源，但是要求资源文件里面的对应资源设置访问为公开

如果没有设置那么将会在 xaml 运行的时候提示

```csharp
System.Windows.Markup.XamlParseException 

在 System.Windows.Markup.StaticExtension 上提供值xxx
```

此时在设计器里面是可以看到绑定成功，只是在运行的时候提示找不到，展开可以看到下面提示

```csharp
无法将 xx.Properties.Resources.xx  StaticExtension 值解析为枚举、静态字段或静态属性
```

解决方法是在 Resource.resx 里面的访问权限从 internal 修改为 public 就可以

## 判断 WPF 程序使用管理员权限运行

引用命名空间，复制下面代码，然后调用 IsAdministrator 方法，如果返回 true 就是使用管理员权限运行

```csharp
using System.Security.Principal;

        public static bool IsAdministrator()
        {
            WindowsIdentity current = WindowsIdentity.GetCurrent();
            WindowsPrincipal windowsPrincipal = new WindowsPrincipal(current);
            //WindowsBuiltInRole可以枚举出很多权限，例如系统用户、User、Guest等等
            return windowsPrincipal.IsInRole(WindowsBuiltInRole.Administrator);
        }

```

[C# 判断软件是否是管理员权限运行 - 除却猩猩不是猿 - CSDN博客](https://blog.csdn.net/zuoyefeng1990/article/details/62224387 )

## 注册全局事件

如果需要注册一个类型的全局事件，如拿到 TextBox 的全局输入，那么可以使用下面代码

```csharp
EventManager.RegisterClassHandler(typeof(TextBox), TextBox.KeyDownEvent, new RoutedEventHandler(方法));
```

## 高版本的 WPF 引用低版本类库导致无法启动

如果在一个 .net 4.0 的 WPF 程序引用一个 .net 2.0 的库，那么就会让程序无法运行，解决方法添加`useLegacyV2RuntimeActivationPolicy`

打开 app.config 添加 `useLegacyV2RuntimeActivationPolicy="true"` 在 startup 元素

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
         var temp = new delegate(){ Func };
         c(temp);
    }

```

于是在函数完就把temp放到gc在调用时找不到委托。

一个好的做法


```csharp
    private static void Func(){}
    private delegate Temp { get; } = new delegate(){Func};
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

[https://stackoverflow.com/a/47952098/6116637](https://stackoverflow.com/a/47952098/6116637)

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

使用 `&#10;` 就可以换行

[win10 uwp 在 xaml 让 TextBlock 换行](https://blog.lindexi.com/post/win10-uwp-%E5%9C%A8-xaml-%E8%AE%A9-textblock-%E6%8D%A2%E8%A1%8C )

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

## 使用 Task ContinueWith 在主线程

在有时候使用 Task 的 Delay 之后想要返回主线程，可以使用 ContinueWith 的方法，请看代码

```csharp
            Task.Delay(TimeSpan.FromSeconds(5)).ContinueWith
            (
                _ => Foo()
                // 如果 Foo 不需要在主线程，请注释下面一段代码
                , TaskScheduler.FromCurrentSynchronizationContext()
            );
```

核心是 TaskScheduler.FromCurrentSynchronizationContext 方法

如果 Foo 不需要在主线程，就可以删除 TaskScheduler.FromCurrentSynchronizationContext 代码

## WPF-数据绑定：日期时间格式

```
{Binding datetime,StringFormat='{}{0:yyyy年MM月dd日 dddd HH:mm:ss}',ConverterCulture=zh-CN}
```

指定ConverterCulture为zh-CN后星期就显示为中文了。

## WPF 第三方DLL 强签名

参见：http://www.cnblogs.com/xjt927/p/5317678.html

## WPF 去掉最大化按钮

通过在窗口添加下面代码

```csharp
ResizeMode="NoResize"
```

窗口就剩下一个关闭同时用户也无法拖动修改窗口大小

## WPF TextBox 全选

在一个按钮点击的时候全选 TextBox 的内容，可以在按钮里面调用 SelectAll 方法

```
textBox.SelectAll();
```

上面代码的 textBox 就是界面写的 TextBox 元素

如果发现调用上面的代码 TextBox 没有全选，可能是 TextBox 没有拿到焦点，可以尝试下面代码

```
textBox.Focus();
textBox.SelectAll();
```

## WPF 获取文本光标宽度

通过 `SystemParameters.CaretWidth` 获取宽度

```csharp
var caretWidth = SystemParameters.CaretWidth;
```

## 获取屏幕可用大小

```csharp
SystemParameters.WorkArea
```

## 设置另一个窗口获取焦点

设置窗口获取焦点不能通过 Focus 设置，这个方法设置的是窗口控件拿到窗口内焦点，需要通过 Activate 方法激活窗口

```csharp
window.Activate();
```

推荐在子窗口关闭之前激活 Owner 解决[关闭模态窗口后，父窗口居然失去焦点跑到了其他窗口的后面的问题 - walterlv](https://blog.walterlv.com/post/fix-owner-window-dropping-down-when-close-a-modal-child-window.html )

详细请看 [SystemParameters.CaretWidth Property](https://docs.microsoft.com/en-us/dotnet/api/system.windows.systemparameters.caretwidth?view=netframework-4.8#System_Windows_SystemParameters_CaretWidth )

[wpf动画——new PropertyPath属性链 - 影天 - 博客园](http://www.cnblogs.com/xwlyun/archive/2012/09/14/2685199.html)

[wpf动画——缓动动画Animation Easing - 影天 - 博客园](http://www.cnblogs.com/xwlyun/archive/2012/09/11/2680579.html)


## WPF ListView 使用 WrapPanel 没有自动换行

原因是没有设置禁用 ListView 的水平滚动

```xml
<ListView ScrollViewer.HorizontalScrollBarVisibility="Disabled">
  <ListView.ItemsPanel>
    <ItemsPanelTemplate>
      <WrapPanel Orientation="Horizontal" />
    </ItemsPanelTemplate>
  </ListView.ItemsPanel>
</ListView>
```

## 通用路由事件定义

因为没有模版创建，还是写一下，方便抄代码

```csharp
        public static readonly RoutedEvent LindexiEvent = EventManager.RegisterRoutedEvent("Lindexi",
            RoutingStrategy.Bubble, typeof(EventHandler<LindexiRoutedEventArgs>), typeof(Owner));

        public event EventHandler<LindexiRoutedEventArgs> Lindexi
        {
            add { AddHandler(LindexiEvent, value); }
            remove { RemoveHandler(LindexiEvent, value); }
        }
```

全部代码

```csharp
    public class Owner : UIElement
    {
        public static readonly RoutedEvent LindexiEvent = EventManager.RegisterRoutedEvent("Lindexi",
            RoutingStrategy.Bubble, typeof(EventHandler<LindexiRoutedEventArgs>), typeof(Owner));

        public event EventHandler<LindexiRoutedEventArgs> Lindexi
        {
            add { AddHandler(LindexiEvent, value); }
            remove { RemoveHandler(LindexiEvent, value); }
        }

        public void RaiseLindexiEvent()
        {
            RaiseEvent(new LindexiRoutedEventArgs(LindexiEvent, this));
        }
    }

    public class LindexiRoutedEventArgs : RoutedEventArgs
    {
        /// <inheritdoc />
        public LindexiRoutedEventArgs()
        {
        }

        /// <inheritdoc />
        public LindexiRoutedEventArgs(RoutedEvent routedEvent) : base(routedEvent)
        {
        }

        /// <inheritdoc />
        public LindexiRoutedEventArgs(RoutedEvent routedEvent, object source) : base(routedEvent, source)
        {
        }
    }
```

监听

```csharp
            xx.AddHandler(Owner.LindexiEvent, new EventHandler<LindexiRoutedEventArgs>((o, args) =>
            {

            }));
```

