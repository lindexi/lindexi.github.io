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