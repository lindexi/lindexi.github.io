# win10 UWP 剪贴板 Clipboard

本文告诉大家如何使用剪贴板 Clipboard 读取剪贴板和放内容在剪贴板

<!--more-->
<!-- CreateTime:2018/9/30 18:54:23 -->


<div id="toc"></div>

在 win10 UWP 可以通过 `Windows.ApplicationModel.DataTransfer.Clipboard` 是剪贴板，剪贴板可以用来与其他程序的通信，目标程序可以使用 UWP 程序也可以使用以前的程序。

下面告诉大家如何去设置和获取剪贴板的内容。

剪贴板的存放使用的是`DataPackage`，里面提供一些默认的方法，因为`DataPackage`在放数据前需要指定数据的id，也就是一个字符串。实际DataPackage可以放任意类型。下面告诉大家如何设置文本。

## 设置文本

在UWP把字符串添加到剪贴板使用代码很少。

第一个创建 DataPackage，无论添加图片还是什么都是使用 DataPackage ，只有他可以放到剪贴板。

```csharp
 DataPackage dataPackage = new DataPackage();
```

然后把文本设置 DataPackage ，因为剪贴板只能设置 DataPackage ，所以即使添加字符串，也是使用 DataPackage

```csharp
            dataPackage.SetText("文本");
            Clipboard.SetContent(dp);
```

设置图片的方法也是相同。

但是 75351663 大神说，设置之前需要清空剪贴板，不然之前数据成为垃圾内存，我自己没有去试，但是加一句代码也没什么，建议设置之前清空。

如果需要设置任意类型，请使用`SetData`，这时设置类型可以是随意。但是可能设置不成功。

```csharp
                var data = new DataPackage();
                    data.SetData("字符串","内容");
```

## 获取文本

如果需要获取文本，一般在开始都判断是否包含文本。一般在需要拿到文本之前，使用`Contains`判断是否存在某个类型，而`Contains`的参数是字符串，可以使用`StandardDataFormats`来获得这些字符串。因为 UWP 的剪贴板是系统的，所以需要兼容以前的软件，以前的软件对剪贴板使用是传入字符串和内容，所以就需要使用字符串去拿。微软封装好了一些内容，这样在设置、获取内容就不需要自己指定字符串和通过内容到本地类型。

检查剪贴板包含文本

```csharp
DataPackageView con = Windows.ApplicationModel.DataTransfer.Clipboard.GetContent();
if (con.Contains(StandardDataFormats.Text))
```

```csharp
            DataPackageView con = Windows.ApplicationModel.DataTransfer.Clipboard.GetContent();
            string str = string.Empty;
            if (con.Contains(StandardDataFormats.Text))
            {
                str = await con.GetTextAsync();
            }
```

## 获取图片

如果只是需要获得图片并且显示图片，可以使用下面的代码

```csharp
        private async Task SetClipimage(DataPackageView data)
        {
            RandomAccessStreamReference file = await data.GetBitmapAsync();
            BitmapImage image = new BitmapImage();
            await image.SetSourceAsync(await file.OpenReadAsync());
            Image.Source = image;
        }
```

但是需要把剪贴板的图片写入到本地，那么需要使用下面的代码

```csharp
            if (con.Contains(StandardDataFormats.Bitmap))
            {
                RandomAccessStreamReference img = await con.GetBitmapAsync();
                var imgstream = await img.OpenReadAsync();
                BitmapImage bitmap = new BitmapImage();
                bitmap.SetSource(imgstream);

                Windows.UI.Xaml.Media.Imaging.WriteableBitmap src = new Windows.UI.Xaml.Media.Imaging.WriteableBitmap(bitmap.PixelWidth, bitmap.PixelHeight);
                src.SetSource(imgstream);

                Windows.Graphics.Imaging.BitmapDecoder decoder = await Windows.Graphics.Imaging.BitmapDecoder.CreateAsync(imgstream);
                Windows.Graphics.Imaging.PixelDataProvider pxprd = await decoder.GetPixelDataAsync(Windows.Graphics.Imaging.BitmapPixelFormat.Bgra8, Windows.Graphics.Imaging.BitmapAlphaMode.Straight, new Windows.Graphics.Imaging.BitmapTransform(), Windows.Graphics.Imaging.ExifOrientationMode.RespectExifOrientation, Windows.Graphics.Imaging.ColorManagementMode.DoNotColorManage);
                byte[] buffer = pxprd.DetachPixelData();

                str = "image";
                StorageFolder folder = await _folder.GetFolderAsync(str);

                StorageFile file = await folder.CreateFileAsync(DateTime.Now.Year.ToString() + DateTime.Now.Month.ToString() + DateTime.Now.Day.ToString() + DateTime.Now.Hour.ToString() + DateTime.Now.Minute.ToString() + ".png", CreationCollisionOption.GenerateUniqueName);

                using (var fileStream = await file.OpenAsync(FileAccessMode.ReadWrite))
                {
                    var encoder = await Windows.Graphics.Imaging.BitmapEncoder.CreateAsync(Windows.Graphics.Imaging.BitmapEncoder.PngEncoderId, fileStream);
                    encoder.SetPixelData(Windows.Graphics.Imaging.BitmapPixelFormat.Bgra8, Windows.Graphics.Imaging.BitmapAlphaMode.Straight, decoder.PixelWidth, decoder.PixelHeight, decoder.DpiX, decoder.DpiY, buffer);
                    await encoder.FlushAsync();
                }
            }
```

## 获取文件

```csharp
 if (con.Contains(StandardDataFormats.StorageItems))
            {
                var filelist = await con.GetStorageItemsAsync();
                foreach (StorageFile t in filelist)
                {
                    
                }
            }
```

IStorageItem 转 StorageFile

```csharp
                     if (t.IsOfType(StorageItemTypes.File))
                    {
                        StorageFile storageFile = storageItem as StorageFile;  
                    }
```

在以前的软件，可以用过剪贴板获得任意内容，而uwp只能获得有限的内容，如果需要获得以前软件的特殊内容，那么请使用`GetDataAsync`，这个方法不建议用。

参考：

http://www.cnblogs.com/tcjiaan

http://www.cnblogs.com/chengxingliang/archive/2013/01/21/2857718.html

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

