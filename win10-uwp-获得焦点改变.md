# win10 uwp 获得焦点改变

本文讲的是当我们应用失去焦点时，我们获得事件，当我们应用获得焦点，同样获得事件。同时，在应用不可以见时，我们也可以获得。

<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>

![这里写图片描述](http://img.blog.csdn.net/20160923204915589)

上面一张图，开始是应用启动，获得焦点，应用显示。然后我们打开另一个应用，切换，这时我们应用没有焦点。

然后我们用鼠标点击应用，我们应用获得焦点，这时显示鼠标点击获得焦点。

我们使用Alt+tab，切换出去别的软件，然后使用Alt+tab切换回，可以看到获得焦点。

然后点击任务栏，把应用最小化，这时我们的应用不显示，因为点开他就显示，我就使用字符串`+`这样我们的应用就可以获得失去焦点和从哪获得焦点、应用不显示。

我们应用有两个事件，一个是`Window.Current.Activated`一个`Window.Current.VisibilityChanged`，我们可以通过`Window.Current.Activated`来知道，我们的应用得到焦点。

UWP应用获得焦点可以使用`Window.Current.Activated`，事件参数有几个，`WindowActivatedEventArgs e`，`CoreWindowActivationState`有

 - CodeActivated 
   从操作系统拿到焦点，这个就是我们使用Alt+tab，不是鼠标点击的获得焦点。我们这里，用鼠标点击任务栏打开应用，是操作系统给应用焦点。
  

 - PointerActivated
   鼠标点击获得焦点，用鼠标点击应用。
 
 - Deactivated
   没有焦点。

我用简单的字符串绑定到View，我们可以看到，我们应用失去焦点和获得焦点。

``` C#
        private void Current_Activated(object sender, WindowActivatedEventArgs e)
        {
            if (e.WindowActivationState == CoreWindowActivationState.CodeActivated)
            {
                ActivateReminder = "获得焦点\r\n";
            }
            else if (e.WindowActivationState == CoreWindowActivationState.PointerActivated)
            {
                ActivateReminder = "鼠标点获得\r\n";
            }
            else if (e.WindowActivationState == CoreWindowActivationState.Deactivated)
            {
                ActivateReminder = "没焦点\r\n";
            }
        }

```

我们可以用`Window.Current.VisibilityChanged`获得我们应用显示，参数`VisibilityChangedEventArgs e`如果`e.Visible==true`就是显示。

我们把应用最小化，应用就会不显示。

参见：http://grogansoft.com/blog/?p=1269

## 如何判断应用是否获得焦点

有时候需要判断应用是否获得焦点，我的图床软件判断当前应用有焦点就自动复制。

但是不可以通过 `Window.Current.CoreWindow.Visible` 判断窗口是否获得焦点，这个值判断是窗口是否最小或，在手机可以使用这个，但是在pc不可以，因为可能我是打开其他的软件。

为了在UWP 判断窗口是否获得焦点，简单的方法是使用上面的代码在失去焦点获得，从而设置一个本地值，让他知道是否获得焦点。


```csharp
         private void Current_Activated(object sender, WindowActivatedEventArgs e)
        {
            _visibility = e.WindowActivationState != CoreWindowActivationState.Deactivated;
        }


         private bool _visibility = true;

```
但是还需要加上 当窗口的可见变化时的


```csharp
                Window.Current.VisibilityChanged += Current_VisibilityChanged;
        private void Current_VisibilityChanged(object sender, VisibilityChangedEventArgs e)
        {
            _visibility = e.Visible;
        }
```
如果是在一个页面或一个控件使用上面的代码，注意，因为函数被加到一个静态的值，所以这个控件或page不会被释放，因为一直有引用。
如果经常进行导航，而且都是新建页面，那么会创建很多页面。

这些页面不会释放，于是内存占用越来越大，所以需要进行释放，方法是在视觉树取消时，从静态值删除函数。

```csharp
         private void Unloaded(object sender, RoutedEventArgs e)
        {
            Window.Current.VisibilityChanged -= Current_VisibilityChanged;
            Window.Current.Activated -= Current_Activated;
        }
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
