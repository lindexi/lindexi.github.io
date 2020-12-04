# WPF 已知问题 窗口在屏幕外创建将不会刷新渲染

本文告诉大家一个 WPF 的已知问题，如果窗口在创建的时候，设置在屏幕外，那这个窗口将不会进行实际的渲染，将这个窗口从屏幕外移到屏幕内的时候，将会出现窗口内容的一次闪烁。换句话说就是存在窗口内容的重渲染

<!--more-->
<!-- CreateTime:2020/10/7 12:54:38 -->



什么是窗口在屏幕外创建？简单说法就是窗口不在屏幕内，如窗口的 Top 或 Left 太大或太小等，如下面代码创建窗口，而我的屏幕没有那么大，因此窗口就显示在我的屏幕外

```csharp
        private async void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var window = new Window()
            {
                Background = Brushes.Gray,
                Height = 200,
                Width = 200,
                Top = 100000 // 手动高亮，我的屏幕没有那么大
            };

            window.Show();

            await Dispatcher.Yield();
            window.Top = 200;
        }
```

运行如上面代码，可以看到在 `window.Top = 200;` 调用的时候，将窗口从屏幕外移动到屏幕内时，窗口的背景从白色切换为灰色。如果你看不到，只能证明你的电脑性能太好了，换个渣设备试试

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/d8516f04/NerehebunaywarRoheeyeekularu ) 欢迎小伙伴访问

如果你将设置窗口的位置，也就是设置 `Top = 100000 // 手动高亮，我的屏幕没有那么大` 放在 Show 之后的一个 Render 内，那么依然窗口闪烁

```csharp
        private async void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var window = new Window()
            {
                Background = Brushes.Gray,
                Height = 200,
                Width = 200,
            };

            window.Show();

            // 设置为 Render 优先级，那么用户将看到窗口显示然后消失
            //await Dispatcher.Yield(DispatcherPriority.Render);

            // 设置为 Normal 优先级，那么窗口依然没有渲染
            await Dispatcher.Yield(DispatcherPriority.Normal);

            window.Top = 100000; // 手动高亮，我的屏幕没有那么大

            await Task.Delay(TimeSpan.FromSeconds(1));
            window.Top = 200;
        }
```

如果放在 Render 外，此时用户就可以先看到窗口显示在屏幕上，然后窗口再消失，依然可以看到窗口闪烁。但是如果窗口能足够卡，也许此时很多设备都是看不到这个创建的窗口

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
