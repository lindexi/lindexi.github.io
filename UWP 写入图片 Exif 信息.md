# UWP 写入图片 Exif 信息

本文告诉大家如何在 UWP 中，保存图片的时候，写入 Exif 信息，也就是如照片的 相机型号 制造商 光圈值等信息的写入

<!--more-->
<!-- CreateTime:2021/7/8 18:52:02 -->

<!-- 发布 -->

在 UWP 中，保存图片或照片需要用到图片编码器，在使用编码器写入前可以设置编码器写入图片的属性，此时就可以包含了 Exif 信息。关于啥是 Exif 信息，还请自行百度

不同的图片格式可以支持的 Exif 信息范围不相同，咱以下使用 jpg 图片作为例子。如果大家切换为其他图片格式，还请自行测试一下

在创建编码器可以在构造函数传入参数，通过参数设置一些 Exif 信息，如质量信息。下面代码在创建时传入质量信息

```csharp
                BitmapPropertySet propertySet = new BitmapPropertySet();
                BitmapTypedValue qualityValue = new BitmapTypedValue(0.77, PropertyType.Single);
                propertySet.Add("ImageQuality", qualityValue);

                var encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.JpegEncoderId, pngStream.AsRandomAccessStream(), propertySet);
```

上面代码的 pngStream 是一个文件，用于写入图片，这部分代码不是本文重点，如果要获取全部的代码，还请到本文最后获取代码

在创建完成编码器之后，依然可以再次设置图片信息，通过调用 encoder.BitmapProperties.SetPropertiesAsync 方法进行设置

如以下代码，设置作者信息

```csharp
                var encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.JpegEncoderId, pngStream.AsRandomAccessStream(), propertySet);

                propertySet = new BitmapPropertySet();
                // 作者
                propertySet.Add("System.Author", new BitmapTypedValue("lindexi", PropertyType.String));
                await encoder.BitmapProperties.SetPropertiesAsync(propertySet);
```

写入之后，可以右击图片文件的属性，进入详细信息。在详细信息里面可以看到图片的信息

以上有一个问题是，能写入属性有哪些，写入的类型是什么？这些可以从 [官方文档](https://docs.microsoft.com/en-us/windows/win32/properties/windows-properties-system?WT.mc_id=WD-MVP-5003260) 获取

如官方文档里面说写入相机型号的描述如下

```
propertyDescription
   name = System.Photo.CameraManufacturer
   shellPKey = PKEY_Photo_CameraManufacturer
   formatID = 14B81DA1-0135-4D31-96D9-6CBFC9671A99
   propID = 271
   SearchInfo
      InInvertedIndex = true
      IsColumn = true
   typeInfo
      type = String
```

以上的含义就是写入的 Key 是 `System.Photo.CameraManufacturer` 要求传入的类型是 PropertyType.String 字符串，根据这个即可了解如何写以上的代码。如写入相机型号的描述等代码如下

```csharp
                var encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.JpegEncoderId, pngStream.AsRandomAccessStream(), propertySet);

                // https://docs.microsoft.com/en-us/windows/win32/properties/windows-properties-system?WT.mc_id=WD-MVP-5003260
                propertySet = new BitmapPropertySet();
                // 作者
                propertySet.Add("System.Author", new BitmapTypedValue("lindexi", PropertyType.String));
                // 相机型号
                propertySet.Add("System.Photo.CameraModel", new BitmapTypedValue("lindexi", PropertyType.String));
                // 制造商
                propertySet.Add("System.Photo.CameraManufacturer", new BitmapTypedValue("lindexi manufacturer", PropertyType.String));
                // 光圈值 System.Photo.FNumberNumerator/System.Photo.FNumberDenominator
                propertySet.Add("System.Photo.FNumberNumerator", new BitmapTypedValue(1, PropertyType.UInt32));
                propertySet.Add("System.Photo.FNumberDenominator", new BitmapTypedValue(10, PropertyType.UInt32));

                await encoder.BitmapProperties.SetPropertiesAsync(propertySet);
```

下面代码是在加载页面，然后进行截图，保存截图到本地文件的代码

```csharp
        public MainPage()
        {
            this.InitializeComponent();

            Loaded += MainPage_Loaded;
        }

        private async void MainPage_Loaded(object sender, RoutedEventArgs e)
        {
            await Task.Delay(TimeSpan.FromSeconds(1));

            var renderTargetBitmap = new RenderTargetBitmap();
            await renderTargetBitmap.RenderAsync(Grid);

            var pngFile = await ApplicationData.Current.TemporaryFolder.CreateFileAsync(Path.GetRandomFileName() + ".jpg");
            using (var pngStream = await pngFile.OpenStreamForWriteAsync())
            {
                BitmapPropertySet propertySet = new BitmapPropertySet();
                BitmapTypedValue qualityValue = new BitmapTypedValue(0.77, PropertyType.Single);
                propertySet.Add("ImageQuality", qualityValue);

                var encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.JpegEncoderId, pngStream.AsRandomAccessStream(), propertySet);

                // https://docs.microsoft.com/en-us/windows/win32/properties/windows-properties-system?WT.mc_id=WD-MVP-5003260
                propertySet = new BitmapPropertySet();
                // 作者
                propertySet.Add("System.Author", new BitmapTypedValue("lindexi", PropertyType.String));
                // 相机型号
                propertySet.Add("System.Photo.CameraModel", new BitmapTypedValue("lindexi", PropertyType.String));
                // 制造商
                propertySet.Add("System.Photo.CameraManufacturer", new BitmapTypedValue("lindexi manufacturer", PropertyType.String));
                // 光圈值 System.Photo.FNumberNumerator/System.Photo.FNumberDenominator
                propertySet.Add("System.Photo.FNumberNumerator", new BitmapTypedValue(1, PropertyType.UInt32));
                propertySet.Add("System.Photo.FNumberDenominator", new BitmapTypedValue(10, PropertyType.UInt32));

                await encoder.BitmapProperties.SetPropertiesAsync(propertySet);

                var pixelBuffer = await renderTargetBitmap.GetPixelsAsync();

                var softwareBitmap = SoftwareBitmap.CreateCopyFromBuffer(pixelBuffer, BitmapPixelFormat.Bgra8, renderTargetBitmap.PixelWidth, renderTargetBitmap.PixelHeight);
                encoder.SetSoftwareBitmap(softwareBitmap);


                await encoder.FlushAsync();

                softwareBitmap.Dispose();
            }

            await Launcher.LaunchFolderAsync(ApplicationData.Current.TemporaryFolder);
        }
```

本文代码可以到 [写入图片Exif信息.7z-CSDN](https://download.csdn.net/download/lindexi_gd/20089118) 下载

本文上面代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/acdca3ea99682d6549cf2622fb96685531ab9ded/KechinabeleenalLechefahar) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/acdca3ea99682d6549cf2622fb96685531ab9ded/KechinabeleenalLechefahar) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin acdca3ea99682d6549cf2622fb96685531ab9ded
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 KechinabeleenalLechefahar 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
