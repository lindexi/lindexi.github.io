# win10 uwp 获得缩略图

有时候需要获得文件或视频的缩略图。

本文提供两个方法，用于获得文件的缩略图和截取视频指定时间的显示图片。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


## 文件缩略图

如果有一个文件需要获得缩略图，可以使用 `GetThumbnailAsync` 或 `GetScaledImageAsThumbnailAsync` ，就可以获得。代码请看下面：

```csharp
            FileOpenPicker openPicker = new FileOpenPicker();
            openPicker.FileTypeFilter.Add(".mp4");

            StorageFile file = await openPicker.PickSingleFileAsync();
            var thumbnail =  await file.GetScaledImageAsThumbnailAsync(ThumbnailMode.VideosView);
            BitmapImage bitmapImage = new BitmapImage();
            InMemoryRandomAccessStream randomAccessStream = new InMemoryRandomAccessStream();
            await RandomAccessStream.CopyAsync(thumbnail, randomAccessStream);
            randomAccessStream.Seek(0);
            bitmapImage.SetSource(randomAccessStream);
            Image.Source = bitmapImage;
```

可以看到 `GetScaledImageAsThumbnailAsync` 的参数提供多种方式进行截图，如果知道了文件的类型，就可以使用对应的方式进行显示。需要知道的是 thumbnail 得到的是一个流，就需要把他转换为 BitmapImage 显示。

我接下来获取文件夹内所有文件的缩略图显示出来

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20176272002.jpg)

首先是界面代码

```csharp
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Grid.RowDefinitions>
            <RowDefinition Height="587*"/>
            <RowDefinition Height="auto"/>
        </Grid.RowDefinitions>
        <Grid>
            <ListView Margin="10,10,10,10" ItemsSource="{x:Bind Ruqya}" HorizontalAlignment="Stretch">
                <ListView.ItemsPanel>
                    <ItemsPanelTemplate>
                        <WrapGrid ItemWidth="200"  Orientation="Horizontal" MaximumRowsOrColumns="5"></WrapGrid>
                    </ItemsPanelTemplate>
                </ListView.ItemsPanel>
             <ListView.ItemTemplate>
                 <DataTemplate x:DataType="local:Ruqya">
                     <Grid>
                         <Image Width="100" Height="100" Source="{x:Bind Image}"></Image>
                     </Grid>
                 </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>
        </Grid>
        <Grid Grid.Row="1">
            <Button Margin="10,10,10,10" Content="确定" Click="Button_OnClick"></Button>
        </Grid>
    </Grid>

```

界面是有一个 ListView 和一个 Button ，点击 Button 就会执行从一个文件夹获取所有文件，拿到缩略图。

但是还需要一个类，用于界面显示图片

```csharp
    public class Ruqya
    {
        public Ruqya(BitmapImage image)
        {
            Image = image;
        }

        public BitmapImage Image { get; set; }
    }
```

因为这时的 Image 不会修改，就不需要继承通知

按钮点击的代码，就是核心，需要从文件夹获得文件。

```csharp

        public ObservableCollection<Ruqya> Ruqya { get; set; } = new ObservableCollection<Ruqya>();

        private async void Button_OnClick(object sender, RoutedEventArgs e)
        {
            FolderPicker pick = new FolderPicker();
            pick.FileTypeFilter.Add(".txt");
            var folder = await pick.PickSingleFolderAsync();
            if (folder == null)
            {
                return;
            }

            Ruqya.Clear();

            //获取文件夹所有文件的缩略图

            foreach (var temp in await folder.GetFilesAsync())
            {
                var thumbnail = await temp.GetThumbnailAsync(ThumbnailMode.SingleItem);
                BitmapImage bitmapImage = new BitmapImage();
                InMemoryRandomAccessStream randomAccessStream = new InMemoryRandomAccessStream();
                await RandomAccessStream.CopyAsync(thumbnail, randomAccessStream);
                randomAccessStream.Seek(0);
                bitmapImage.SetSource(randomAccessStream);
                Ruqya.Add(new Ruqya(bitmapImage));
            }
        }
```

所有代码就这一点，相信没有看不懂。

接下来告诉大家如何获得视频的小图

## 视频小图

如果需要获得视频的某一个页面，那么可以使用下面代码，首先是获得视频文件，计算指定时间的视频截图，这时不需要进行播放视频就可以从文件直接获得指定时间的显示图片。

```csharp
            FileOpenPicker openPicker = new FileOpenPicker();
            openPicker.FileTypeFilter.Add(".mp4");

            StorageFile file = await openPicker.PickSingleFileAsync();
            var thumbnail = await GetThumbnailAsync(file);
            BitmapImage bitmapImage = new BitmapImage();
            InMemoryRandomAccessStream randomAccessStream = new InMemoryRandomAccessStream();
            await RandomAccessStream.CopyAsync(thumbnail, randomAccessStream);
            randomAccessStream.Seek(0);
            bitmapImage.SetSource(randomAccessStream);
            Image.Source = bitmapImage;

          
          public async Task<IInputStream> GetThumbnailAsync(StorageFile file)
        {
            var mediaClip = await MediaClip.CreateFromFileAsync(file);
            var mediaComposition = new MediaComposition();
            var time=5000; 获取那一时间的页面
            mediaComposition.Clips.Add(mediaClip);
            return await mediaComposition.GetThumbnailAsync(
                TimeSpan.FromMilliseconds(time), 0, 0, VideoFramePrecision.NearestFrame);
        }
```

这样就可以获得指定时间的页面，因为得到图像是 IInputStream ，所以需要把他转为 bitmapImage ，这样才可以设置为图片。这个方法只需要传入视频的文件，大法支持很多个视频类型，只要有系统解析的，就可以支持，暂时我还不知道他支持的是哪些文件。

接下来就是做下面的软件，在播放视频的时候，拖动进度条，就会显示对应的视频缩略图，如拖到指定时间，就显示这一时间的视频缩略图

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017%25E5%25B9%25B46%25E6%259C%258827%25E6%2597%25A5%25202929.gif)

<!-- ![](http://wx4.sinaimg.cn/large/005VX1Eagy1fh027j7qutg30z10nyu11.gif) -->

<!-- ![](http://wx1.sinaimg.cn/mw690/005VX1Eagy1fh026g7k0ej30z10nyag1.jpg) -->

首先是界面代码，可以看到界面就一个播放和一个进度条

```csharp
           <MediaElement x:Name="MediaElement" Margin="10,10,10,10" Stretch="None"
                         HorizontalAlignment="Stretch" VerticalAlignment="Stretch"></MediaElement>
            
           <Grid VerticalAlignment="Bottom">
               <Slider x:Name="Slider" Margin="10,10,10,10" PointerReleased="UIElement_OnPointerReleased" PointerPressed="UIElement_OnPointerReleased"></Slider>
           </Grid>
```

后台代码很多都使用上面的代码，需要知道的有两个代码。第一个是`OnPointerReleased` 需要添加在构造函数，因为滑动进度会吃了 PointerReleased ，如果不把下面的代码写入，就使用上面界面的代码，可以看到点击的函数不会进去。但是如果加了下面的代码，就可以获得点击的事件。在 UWP 没有区分触摸和鼠标点击，都使用 Pointer 来说是点击结束或者点击。

```csharp
            Slider.AddHandler(PointerReleasedEvent, new PointerEventHandler(UIElement_OnPointerReleased), true);

```

需要知道的第二个就是如何进行播放视频，因为上面代码已经从可以选到文件，于是就可以使用从文件播放的方式，让播放器使用文件。代码很简单，对于需要的 MIME 可以忽略，直接给空。如果对他给空，当然要求文件是 mp4 。如果文件是其他的，建议不要给空，播放器解析也许出错。

```csharp
            MediaElement.SetSource(await file.OpenAsync(FileAccessMode.Read), "");

            如果文件类型不是 mp4 ，请用下面代码
            MediaElement.SetSource(await file.OpenAsync(FileAccessMode.Read), file.ContentType); 
```

在点击进度条，就可以获得当前的值，然后计算在视频中的时间，通过这个时间，进行截图。

```csharp
            var slider = (Slider) sender;
            var n = slider.Value / slider.Maximum;
            n = MediaElement.NaturalDuration.TimeSpan.TotalMilliseconds * n;
```
获取视频总时间可以使用 NaturalDuration ，我需要把他转换时间，使用的代码同样很简单

获得需要的时间，就可以使用上面代码进行截图，其中 File 就是刚才选的文件，如果已经播放了，实际还有其它的方法，但是本文说的是文件截图。

```csharp
            var thumbnail = await GetThumbnailAsync(File, n);
            BitmapImage bitmapImage = new BitmapImage();
            InMemoryRandomAccessStream randomAccessStream = new InMemoryRandomAccessStream();
            await RandomAccessStream.CopyAsync(thumbnail, randomAccessStream);
            randomAccessStream.Seek(0);
            bitmapImage.SetSource(randomAccessStream);
```

好像代码已经完成了，但是在哪里显示？为了可以显示，这里使用 Flyout ，给他一个图片控件，用于显示。

```csharp
            Flyout flyout = new Flyout();
            var image = new Image()
            {
                Width = 100,
                Height = 100,
                Stretch = Stretch.Fill,
                Margin = new Thickness(0)
            };
            flyout.Content = image;
            image.Source = bitmapImage;

            flyout.ShowAt(slider);
```

去掉 flyout 背景很简单，我就不说啦，于是所有代码都写在这，不要找我要工程

```csharp
        private async void UIElement_OnPointerReleased(object sender, PointerRoutedEventArgs e)
        {
            var slider = (Slider) sender;
            var n = slider.Value / slider.Maximum;
            n = MediaElement.NaturalDuration.TimeSpan.TotalMilliseconds * n;

            var thumbnail = await GetThumbnailAsync(File, n);
            BitmapImage bitmapImage = new BitmapImage();
            InMemoryRandomAccessStream randomAccessStream = new InMemoryRandomAccessStream();
            await RandomAccessStream.CopyAsync(thumbnail, randomAccessStream);
            randomAccessStream.Seek(0);
            bitmapImage.SetSource(randomAccessStream);

            Flyout flyout = new Flyout();
            var image = new Image()
            {
                Width = 100,
                Height = 100,
                Stretch = Stretch.Fill,
                Margin = new Thickness(0)
            };
            flyout.Content = image;
            image.Source = bitmapImage;

            flyout.ShowAt(slider);
        }

        private async Task<IInputStream> GetThumbnailAsync(StorageFile file, double time)
        {
            var mediaClip = await MediaClip.CreateFromFileAsync(file);
            var mediaComposition = new MediaComposition();
            mediaComposition.Clips.Add(mediaClip);
            return await mediaComposition.GetThumbnailAsync(
                TimeSpan.FromMilliseconds(time), 0, 0, VideoFramePrecision.NearestFrame);
        }
```

感谢 [李继龙](https://kljzndx.github.io/My-Blog/)

参见：https://stackoverflow.com/a/37314446/6116637


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  