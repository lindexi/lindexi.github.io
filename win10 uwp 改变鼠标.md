# win10 uwp 改变鼠标

经常在应用需要修改光标，显示点击、显示输入，但是有些元素不是系统的，那么如何设置鼠标？

本文主要：UWP 设置光标，UWP 移动鼠标
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

## 设置光标

需要写一点代码来让程序比较容易看到，什么光标对于什么。

UWP 设置的光标有些看不懂，直接看不知道他是干什么

在xaml写代码：


```csharp
    
        <StackPanel>
            <TextBlock Margin="10,10,10,10" Text="Hand" PointerEntered="button_OnPointerEntered"></TextBlock>
            <TextBlock Margin="10,10,10,10" Text="Arrow" PointerEntered="button_OnPointerEntered"></TextBlock>
            <TextBlock Margin="10,10,10,10" Text="Cross" PointerEntered="button_OnPointerEntered"></TextBlock>
            <TextBlock Margin="10,10,10,10" Text="Help" PointerEntered="button_OnPointerEntered"></TextBlock>
            <TextBlock Margin="10,10,10,10" Text="Beam" PointerEntered="button_OnPointerEntered"></TextBlock>
        </StackPanel>
```

代码写好了，他可以在鼠标移入TextBlock 进入函数，可以在函数修改UWP 鼠标光标


首先使用`Windows.UI.Xaml.Window.Current.CoreWindow.PointerCursor ` 设置或获取光标。

需要设置光标需要用`Windows.UI.Core.CoreCursor`

他有一些比较多用的类型，下面是他们对于代码

 - Hand 点击

 - Arrow 正常

 - Cross 十字

 - Help 帮助

 - Wait 等待

 - Beam 输入 

于是对应界面


```csharp
    
        private void button_OnPointerEntered(object sender, PointerRoutedEventArgs e)
        {
            string str = (sender as TextBlock)?.Text as string;
            uint n = 1;
            switch (str)
            {
                case "Hand":
                    Window.Current.CoreWindow.PointerCursor = new Windows.UI.Core.CoreCursor(Windows.UI.Core.CoreCursorType.Hand, n);
                    break;
                case "Arrow": Window.Current.CoreWindow.PointerCursor = new Windows.UI.Core.CoreCursor(Windows.UI.Core.CoreCursorType.Arrow, n); break;
                case "Cross": Window.Current.CoreWindow.PointerCursor = new Windows.UI.Core.CoreCursor(Windows.UI.Core.CoreCursorType.Cross, n); break;
                case "Help": Window.Current.CoreWindow.PointerCursor = new Windows.UI.Core.CoreCursor(Windows.UI.Core.CoreCursorType.Help, n); break;
                case "Beam": Window.Current.CoreWindow.PointerCursor = new Windows.UI.Core.CoreCursor(Windows.UI.Core.CoreCursorType.IBeam, n); break;
            }
            
        }

```

试试把代码放到工程，可以看到UWP 光标改变。

如果不知道 n 是什么，我可以说，自定义光标就是使用n，但是复杂。

很少会有需要自己做光标。如果需要自己做，请看[自定义光标](https://blogs.msdn.microsoft.com/devfish/2012/08/01/customcursors-in-windows-8-csharp-metro-applications/)



## 移动鼠标

有时候需要把鼠标移动到一个元素上，UWP 移动鼠标和改变光标一样。

移动鼠标，设置`CoreWindow.PointerPosition`

在界面放一个按钮，点击他，移动鼠标


```csharp
             var p = new Point(Window.Current.Bounds.X + Window.Current.Bounds.Width / 2, Window.Current.Bounds.Y + Window.Current.Bounds.Height / 2);
            Window.Current.CoreWindow.PointerPosition = p;
```

这样移动很简单，移动是屏幕坐标，不是应用坐标，需要对移动加上窗口移动

https://blogs.msdn.microsoft.com/devfish/2012/08/01/customcursors-in-windows-8-csharp-metro-applications/



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 