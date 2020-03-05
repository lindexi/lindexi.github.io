# win10 UWP 显示地图

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

微软自带的地图很简单

第一步引用地图`xmlns:Map="using:Windows.UI.Xaml.Controls.Maps"`

这段代码写在`<Page>`

然后在`Grid` 用 Map 来得到 MapControl
`<Map:MapControl />`

尝试运行

![MapControl](http://img.blog.csdn.net/20151119220140568)

提示 警告：未指定MapServiceToken

在功能选位置功能
![这里写图片描述](http://img.blog.csdn.net/20151119220447500)

要获得位置需要权限

![获得权限](http://img.blog.csdn.net/20151119222529424)

为了获得位置，写一个按钮点击获得位置

MainPage.xaml.cs

```C#
            //需要using Windows.Devices.Geolocation;
            var access = await Windows.Devices.Geolocation.Geolocator.RequestAccessAsync();
            switch (access)
            {
                case GeolocationAccessStatus.Unspecified:
                    //定位未开启提示是否跳转到 设置 页面            
                    return;
                case GeolocationAccessStatus.Allowed:           //允许获取          
                    break;
                case GeolocationAccessStatus.Denied:            //不允许获取位置信息时 给予提示 然后根据情况选择是否跳转到 设置 界面           
                    await Windows.System.Launcher.LaunchUriAsync(new Uri("ms-settings://privacy/location"));
                    return;
                default:
                    break;
            }
            var gt = new Geolocator();
            var position = await gt.GetGeopositionAsync();  //以前的position.Coordinate.Latitude 方法在UWP中已经过时，不再推荐使用    
                                                            //var latiude = position.Coordinate.Latitude;   
            map.Center = position.Coordinate.Point;
            map.ZoomLevel = 10;            
```

因为 map.Center 说的是 Geopoint，王陈染大神说的是`position = await gt.GetGeopositionAsync();`类型是Geoposition，结果错误是出现了
>无法将类型“Windows.Devices.Geolocation.Geoposition”隐式转换为“Windows.Devices.Geolocation.Geopoint”  appButtonBar

正确的代码 `map.Center = position.Coordinate.Point;`

点击就把地图中心设置在用户位置


参考：http://www.wangchenran.com


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。