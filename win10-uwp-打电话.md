#win10 uwp 打电话

UWP可以通过Skype打电话，那么如何通过应用间通讯，很简单使用Launcher。

Skype电话使用`Skype:(电话号)?call` `Skype:(skype id)?call`格式

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=7e49e57fc47834ef429cd0ee15673bde)

我们在电话按钮按下

```

        private async void Button_OnClick(object sender, RoutedEventArgs e)
        {
            Uri url=new Uri(@"Skype:110?call");
            var areSkypeCall = await Windows.System.Launcher.LaunchUriAsync(url);
            if (areSkypeCall)
            {
                //打成功
            }
        }
```

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=5c45af9ae53b84bbd6bf235ad8c1ce58)

打成功是说跳到Skype，用户选择打不打是他的事

如果打电话这么简单，我就不会写这博客，我们还要判断设备。

UWP判断设备可以使用`AnalyticsInfo.VersionInfo.DeviceFamily`，我们判断设备可以选择使用手机的拨号，这个才是真的电话，垃圾Skype

我们需要先引用Windows Mobile Extensions

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=9913efb448c7510d822e4c6dcc570c55)

```

            if (Windows.System.Profile.AnalyticsInfo.VersionInfo.DeviceFamily
               == "Windows.Mobile")
            {
                Windows.ApplicationModel.Calls.PhoneCallManager.ShowPhoneCallUI("110", "警察");
            }
```

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=d42118fd3478528e4688f5f9b3d69255)

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=c0df6586a5d486d0ce7cd9447fd75f59)

好像还是很简单，如果发现有问题可以发邮件给我


参见：http://stackoverflow.com/questions/34777603/uwp-use-skype-to-call-number