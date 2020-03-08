# UWP how to get the touch width

The touch width can help us to make a gorgeous application. This article tells you how to get the touch width from the PointEvent in UWP.

<!--more-->
<!-- CreateTime:2018/11/15 18:49:12 -->


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