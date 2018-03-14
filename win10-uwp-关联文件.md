
# win10 uwp 关联文件

有时候应用需要打开后缀名为x的文件，那么如何从文件打开应用？

<!--more-->



<div id="toc"></div>

首先，需要打开 Package.appxmanifest

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFAMV+BQA28wYAAQAEAK4+AQBmQwIAaOgJAOjZ/2017223193546.jpg)

添加一个功能，需要添加最少有名称，文件类型。

上面的图就是我添加jpg 的方法，我的应用可以打开jpg，我也有一个应用是需要的。

这个就是 UWP图床:[https://www.microsoft.com/store/apps/9nblggh562r2](https://www.microsoft.com/store/apps/9nblggh562r2 )

添加完，打开App.xaml.cs

添加一个函数


```csharp
        protected override void OnFileActivated(FileActivatedEventArgs args)
        {
            base.OnFileActivated(args);
        }
```

需要对他做一点修改


```csharp
        protected override void OnFileActivated(FileActivatedEventArgs args)
        {
            var file = args.Files[0];
            Frame frame = Window.Current.Content as Frame;
            if (frame == null)
            {
                frame = new Frame();
                Window.Current.Content = frame;
            }
            frame.Navigate(typeof(MainPage), file);
             Window.Current.Activate();
 }
```


这里的file可能是空。

页面跳转就是这样，页面传入可以是 StorageFile。

直接显示在 MainPage ，如果需要显示在别的窗口，当然也不是对你有难度。

最好的方法是使用MVVM 参见：[http://lindexi.oschina.io/lindexi/post/win10-uwp-MVVM%E5%85%A5%E9%97%A8/](http://lindexi.oschina.io/lindexi/post/win10-uwp-MVVM%E5%85%A5%E9%97%A8/ )

我没有在博客说如何传参，但是这个对大家也不难。

打开 MainPage.xaml 写一个Image


```xml
            <Image x:Name="Image"></Image>

```

打开 xaml.cs ，把app启动的file显示

```csharp
        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            var file = e.Parameter as StorageFile;
            if (file != null)
            {
                using (var stream = await file.OpenAsync(FileAccessMode.Read))
                {
                    BitmapImage img = new BitmapImage();
                    await img.SetSourceAsync(stream);
                    Image.Source = img;
                }
            }
        }
```

UWP 从文件显示图片很简单，打开放在img就好。

于是打开一个jpg，用这个应用，可以看到，就是简单代码就可以。

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFAMV+BQA28wYAAQAEAK4+AQBmQwIAaOgJAOjZ/%E6%96%87%E4%BB%B6%E6%89%93%E5%BC%80.gif)


一个好看的应用，需要在关联文件加上图片。

看到txt 文件，有一个图片，这个图片，如果应用可以加上一个图片，这个图片就是设置默认应用加上。

但是个人开发者好像不能关联文件。





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。