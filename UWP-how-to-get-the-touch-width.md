
# UWP how to get the touch width

The touch width can help us to make a gorgeous application. This article tells you how to get the touch width from the PointEvent in UWP.

<!--more-->



<!-- csdn -->

Opening the VisualStudio and create an empty UWP application.

We should open the MainPage.xaml and add the background in the Grid to make the Grid can get the PointMove event.

```xml
    <Grid Background="Transparent">

    </Grid>
```

Then we can open the MainPage.xaml.cs to write the code to get the PointerMove event.

```csharp
        public MainPage()
        {
            InitializeComponent();

            Content.PointerMoved += MainPage_PointerMoved;
        }

        private void MainPage_PointerMoved(object sender, PointerRoutedEventArgs e)
        {

        }
```

We can use GetCurrentPoint to get the PointerPoint.

```csharp
        private void MainPage_PointerMoved(object sender, PointerRoutedEventArgs e)
        {
            var point = e.GetCurrentPoint(this);
        }
```

And we can find the [ContactRect](https://docs.microsoft.com/en-us/uwp/api/windows.ui.input.pointerpointproperties.contactrect) in Properties. We can get the touch width from ContactRect.

```csharp
        private void MainPage_PointerMoved(object sender, PointerRoutedEventArgs e)
        {
            var point = e.GetCurrentPoint(this);
            Rect rect = point.Properties.ContactRect;
        }
```

To get the touch width.

```csharp
        private void MainPage_PointerMoved(object sender, PointerRoutedEventArgs e)
        {
            var point = e.GetCurrentPoint(this);
            Rect rect = point.Properties.ContactRect;
            Debug.WriteLine($"Touch rect width={rect.Width},height={rect.Height}");
        }
```

We also can use ContactRectRaw in Properties.

```csharp
        private void MainPage_PointerMoved(object sender, PointerRoutedEventArgs e)
        {
            var point = e.GetCurrentPoint(this);
            Rect rect = point.Properties.ContactRect;
            Debug.WriteLine($"Touch rect width={rect.Width},height={rect.Height}");
            rect = point.Properties.ContactRectRaw;
            Debug.WriteLine($"Touch raw rect width={rect.Width},height={rect.Height}");
        }
```

Try to run the code and touch the application and you can watch the output windows that prints the touch width.




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。