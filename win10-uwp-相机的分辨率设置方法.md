
# win10 uwp 相机的分辨率设置方法

本文来告诉大家如何在 UWP 中修改相机的分辨率设置以及如何使用相机的功能

<!--more-->


<!-- CreateTime:2020/12/31 8:40:27 -->

<!-- 发布 -->

在 UWP 中可以使用 WinRT 提供的 Win10 特有的 API 用来捕获摄像机的内容，支持很多格式的硬件解码，性能会比 Win32 好特别多。我使用了 UWP 版本的和 WPF 基于 DirectShow 的版本进行性能对比

发现在使用 WPF 的版本，在我的设备上，大概 CPU 能到百分之十，而完全没有用到 Video Decode 的 GPU 加速。而在使用 UWP 时，可以发现 CPU 占用小于百分之一，同时可以使用上 Video Decode 的 GPU 加速功能。当然了能否使用 Video Decode 也和相机编码格式相关，我的这个相机只支持 MJPEG 和 YUV 两个格式。本身 YUV 是不需要解码的，只是清晰度比较渣。上面测试使用的是 MJPEG 格式

在开始之前，咱需要了解在 UWP 中开启相机需要哪些步骤？第一步是添加权限，第二步是加上播放器，第三步是加上相机捕获

添加权限的方法是 Package.appxmanifest 里面添加照相机和手机权限，也可以编辑此文件，添加下面代码

```xml
  <Capabilities>
      <Capability Name="internetClient" />
      <DeviceCapability Name="webcam"/>
      <DeviceCapability Name="Microphone"/>
  </Capabilities>
```

接着添加一个相机播放器了，使用 CaptureElement 来作为播放器，在 MainPage 添加下面代码

```xml
      <CaptureElement Name="PreviewControl" Stretch="Uniform"/>
```

接着在 Loaded 事件里面添加捕获相机的代码

```csharp
        private async void MainPage_Loaded(object sender, RoutedEventArgs e)
        {
            try
            {

                _mediaCapture = new MediaCapture();
                await _mediaCapture.InitializeAsync();
            }
            catch (UnauthorizedAccessException)
            {
            	// 没有申请权限，或者用户点击不允许访问相机
                // This will be thrown if the user denied access to the camera in privacy settings
                return;
            }

            try
            {
                PreviewControl.Source = _mediaCapture;
                await _mediaCapture.StartPreviewAsync();
            }
            catch (System.IO.FileLoadException)
            {

            }

        }

        MediaCapture _mediaCapture;
```

此时可以看到的相机使用的编码以及分辨率完全取决于相机，但是咱可以自己来进行设置。相机会告诉系统他支持的所有格式和分辨率和刷新率等，咱需要将这些列举出来，让用户选择

先在 MainPage 添加一个 ComboBox 用于给用户选择

```xml
<ComboBox x:Name="ComboBox" Margin="10,10,10,10" HorizontalAlignment="Right" VerticalAlignment="Top" SelectionChanged="ComboBox_OnSelectionChanged"></ComboBox>
```

在刚才的 MainPage_Loaded 方法里面获取当前相机支持的有哪些格式，将这些作为内容放入到 ComboBox 选项

```csharp
        private async void MainPage_Loaded(object sender, RoutedEventArgs e)
        {
            try
            {
                _mediaCapture = new MediaCapture();
                await _mediaCapture.InitializeAsync();

                try
                {
                    var comboBox = ComboBox;

                    var availableMediaStreamProperties = _mediaCapture.VideoDeviceController.GetAvailableMediaStreamProperties(MediaStreamType.VideoRecord).ToList().OfType<VideoEncodingProperties>()
                        //.OrderByDescending(x => x.Height * x.Width).ThenByDescending(x => x.FrameRate);
                        .ToList() ;

                    // Populate the combo box with the entries
                    foreach (VideoEncodingProperties property in availableMediaStreamProperties)
                    {
                        ComboBoxItem comboBoxItem = new ComboBoxItem();
                        comboBoxItem.Content = property.Width + "x" + property.Height + " " + property.FrameRate + "FPS " + property.Subtype;
                        comboBoxItem.Tag = property;
                        comboBox.Items.Add(comboBoxItem);
                    }
                }
                catch (Exception)
                {

                }
            }
            catch (UnauthorizedAccessException)
            {
                // This will be thrown if the user denied access to the camera in privacy settings
                return;
            }

            try
            {
                PreviewControl.Source = _mediaCapture;
                await _mediaCapture.StartPreviewAsync();
            }
            catch (System.IO.FileLoadException)
            {

            }

        }
```

在用户选择格式的时候，将会触发 ComboBox_OnSelectionChanged 方法，在这个方法里面执行设置相机格式，包括分辨率的方法

```csharp
        private async void ComboBox_OnSelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            try
            {
                var selectedItem = (sender as ComboBox).SelectedItem as ComboBoxItem;
                var encodingProperties = (selectedItem.Tag as VideoEncodingProperties);
                await _mediaCapture.VideoDeviceController.SetMediaStreamPropertiesAsync(MediaStreamType.VideoRecord, encodingProperties);
            }
            catch (Exception)
            {
              
            }
        }
```

在 UWP 中不能直接设置相机的分辨率，而是需要先通过 MediaCapture.VideoDeviceController.GetAvailableMediaStreamProperties 方法获取相机能支持的哪些格式，从里面选出想要的分辨率等设置，通过 MediaCapture.VideoDeviceController.SetMediaStreamPropertiesAsync 设置相机的格式

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/25f6516e756edd518478a88cacdd766c9d00cd32/KucalyabiHuwelberyearni ) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。