
# 三种方式设置特定设备UWP XAML view

开发者可以设置UWP特定设备xaml view，在桌面，手机，Iot，这个对于设置对不同设备的不同屏幕有用。我们可以使用RelativePanel，VisualStateTriggers，但是这样我们的xaml很大，我们在弄的时候觉得想修改一个东西会让我们把全部删了。
为了让我们可以在不同设备使用不同xaml view，我们可以有下面的方式：

<!--more-->



<div id="toc"></div>

在不同设备显示不同背景颜色，文本，在同xaml，开始的页面

<Page  
    x:Class="DeviceFamily.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:DeviceFamily"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="Blue">
        <TextBlock Foreground="White" Text="This is desktop." FontSize="72"
                   VerticalAlignment="Center" HorizontalAlignment="Center" />
    </Grid>
</Page> 

## 新建文件夹DeviceFamily-Type

里面的type是我们的设备，手机：Mobile，桌面：Desktop，IOT

在我们的解决方案新建一个文件夹，我们这里在手机Mobile

![](http://7xqpl8.com1.z0.glb.clouddn.com/16-4-6/21773005.jpg)

我们在新建DeviceFamily-Mobile新建xaml，MainPage

![这里写图片描述](http://res.cloudinary.com/dvi6ot1t1/image/upload/v1438517274/deviceFamilyVS2_g6gazd.jpg)

新建类MainPage

![这里写图片描述](http://res.cloudinary.com/dvi6ot1t1/image/upload/v1438517385/deviceFamilyVS3_zrnft1.jpg)

如果在手机运行，就会使用DeviceFamily-Mobile/MainPage.xaml，如果是其他，就会使用解决方案DeviceFamily/MainPage.xaml
## 在xaml文件加.DeviceFamily-Type
第二种方式在xaml文件加.DeviceFamily-Type，我们创建一个新的xaml，MainPage.DeviceFamily-Mobile.xaml
![这里写图片描述](http://res.cloudinary.com/dvi6ot1t1/image/upload/v1438517871/deviceFamilyVS4_syhdit.jpg)
我们不能使用方法1和2在一个工程。

我们打开mobil会使用MainPage.DeviceFamily-Mobile.xaml

## InitializeComponent重载

添加一个DeviceFamily-Type文件夹在里面写一个xaml会在MainPage.g.i.cs对InitializeComponent重载。

```csharp
public void InitializeComponent(global::System.Uri resourceLocator)  
{
    if (_contentLoaded)
        return;

    _contentLoaded = true;

    if (resourceLocator == null)
    {
        resourceLocator = new global::System.Uri("ms-appx:///MainPage.xaml");
    }
    global::Windows.UI.Xaml.Application.LoadComponent(this, resourceLocator, global::Windows.UI.Xaml.Controls.Primitives.ComponentResourceLocation.Application);
}
```

可以指定一个uri到需要xaml

![这里写图片描述](http://res.cloudinary.com/dvi6ot1t1/image/upload/v1438520919/deviceFamilyVS5_gdgxb8.jpg)

```csharp
public MainPage()  
{
    if (AnalyticsInfo.VersionInfo.DeviceFamily == "Windows.Mobile")
    {
        if (usePrimary)
        {
            InitializeComponent(new Uri("ms-appx:///PrimaryMainPage.xaml", UriKind.Absolute));
        }
        else
        {
            InitializeComponent(new Uri("ms-appx:///SecondaryMainPage.xaml", UriKind.Absolute));
        }
    }
    else
    {
        InitializeComponent();
    }
}
```
使用DeviceFamily指定视图的效果，首先是桌面的MainPage.xaml

![这里写图片描述](http://res.cloudinary.com/dvi6ot1t1/image/upload/v1438521366/desktop_yaxua2.jpg)
我们使用不同颜色放在mobil
![这里写图片描述](http://res.cloudinary.com/dvi6ot1t1/image/upload/v1438521806/mobile1_sin4zt.jpg)
如果使用方式3，我们需要手动在main写我们需要加载，我们有两个xaml
![这里写图片描述](http://res.cloudinary.com/dvi6ot1t1/image/upload/v1438522070/mobile-primary_j8v5fl.jpg)

![这里写图片描述](http://res.cloudinary.com/dvi6ot1t1/image/upload/v1438522070/mobile-secondary_gjihv4.jpg)

单页面触发器

我们可以在一个页面不同设备使用不同的xaml
[WindowsStateTriggers DeviceFamily sample](https://github.com/dotMorten/WindowsStateTriggers/blob/master/src/TestApp/Samples/DeviceFamilySample.xaml)

[https://github.com/igrali/UWP-DeviceFamily](https://github.com/igrali/UWP-DeviceFamily )

[http://www.cnblogs.com/mushroom/p/5080032.html](http://www.cnblogs.com/mushroom/p/5080032.html )

[http://igrali.com/2015/08/02/three-ways-to-set-specific-devicefamily-xaml-views-in-uwp/](http://igrali.com/2015/08/02/three-ways-to-set-specific-devicefamily-xaml-views-in-uwp/ )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。