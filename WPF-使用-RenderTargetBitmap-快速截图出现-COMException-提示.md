
# WPF 使用 RenderTargetBitmap 快速截图出现 COMException 提示

本文告诉大家一个最简单步骤让 RenderTargetBitmap 出现 COMException 提示

<!--more-->


<!-- csdn -->

只需要在界面添加一个 ListView 绑定图片，然后在后台不断刷新列表就可以

```csharp
        <ListView Margin="10,10,10,10" ItemsSource="{Binding DeagernereDechuno}">
            <ListView.ItemTemplate>
                <DataTemplate>
                    <Grid Margin="10,10,10,10">
                        <Image Source="{Binding}"></Image>
                    </Grid>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
```

在后台创建 DeagernereDechuno 列表

```csharp
        public ObservableCollection<ImageSource> DeagernereDechuno { get; set; }=new ObservableCollection<ImageSource>();

```

在 Load 之后调用函数 WarwairJorkasou 不断截图

```csharp
        public MainWindow()
        {
            InitializeComponent();
            DataContext = this;


            Loaded += (s, e) =>
            {
                 WarwairJorkasou();
            };
        }
```

在 WarwairJorkasou 调用循环进行截图，很快就可以看到下面提示

```csharp
 System.Runtime.InteropServices.COMException: MILERR_WIN32ERROR
``` 

异常堆栈

```csharp
System.Runtime.InteropServices.COMException (0x88980003): MILERR_WIN32ERROR (异常来自 HRESULT:0x88980003)
   在 System.Windows.Media.Imaging.RenderTargetBitmap.FinalizeCreation()
   在 System.Windows.Media.Imaging.RenderTargetBitmap..ctor(Int32 pixelWidth, Int32 pixelHeight, Double dpiX, Double dpiY, PixelFormat pixelFormat)
```

截图的代码

```csharp
        private async void WarwairJorkasou()
        {
            var ran = new Random();

            while (true)
            {
                await Task.Delay(10).ContinueWith(_ =>
                {
                    DeagernereDechuno.Clear();
                    var n = ran.Next(int.MaxValue / 10);
                    for (int i = n; i < n + 1000; i++)
                    {
                        try
                        {
                            DrawingVisual drawingVisual = new DrawingVisual();
                            DrawingContext drawingContext = drawingVisual.RenderOpen();

                            var text = new FormattedText(i.ToString(),
                                CultureInfo.GetCultureInfo("zh-cn"),
                                FlowDirection.LeftToRight,
                                new Typeface("Verdana"),
                                36, Brushes.Black);
                            drawingContext.DrawText(text,
                                new Point(0, 0));
                
                            drawingContext.Close();

                            var image = new RenderTargetBitmap((int) text.Width, (int) text.Height, 96, 96, PixelFormats.Pbgra32);
                            image.Render(drawingVisual);

                            DeagernereDechuno.Add(image);
                        }
                        catch (Exception e)
                        {
                            Console.WriteLine(e);
                        }
                    }
                }, TaskScheduler.FromCurrentSynchronizationContext());
            }
        }
```

运行程序大概在 300M 左右就会出现 COMException 提示

```csharp
System.Runtime.InteropServices.COMException (0x88980003): MILERR_WIN32ERROR (Exception from HRESULT: 0x88980003)
```

[RenderTargetBitmap throws COM exception when created too fast: MILERR_WIN32ERROR (Exception from HRESULT: 0x88980003)](https://social.msdn.microsoft.com/Forums/vstudio/en-US/5e9fb69b-7547-4f0b-ba06-ad4211be733d/rendertargetbitmap-throws-com-exception-when-created-too-fast-milerrwin32error-exception-from?forum=wpf )

代码请看 [https://github.com/dotnet-campus/wpf-issues/tree/master/RenderTargetBitmapThrowsCOMExceptionWhenCreatedTooFast](https://github.com/dotnet-campus/wpf-issues/tree/master/RenderTargetBitmapThrowsCOMExceptionWhenCreatedTooFast )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。