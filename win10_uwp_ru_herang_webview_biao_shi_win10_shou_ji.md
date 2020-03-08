# win10 uwp 如何让WebView标识win10手机

本文主要：如何让WebView访问的网页识别为手机.

当然这句话我说不好，换个，如何让 WebView 识别为手机。

上面两句话都是错的，因为是服务器识别，不是网页，第二句话应该是让服务器而不是 WebView 。为什么这样写是因为有大神在群里问这个，他这样说，我这样写希望大家能在搜索看到。当然本文发在csdn和win10.me，其他地方是没有发的，不过我的gitbook.io还是有的。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

如何让WebView识别手机，其实很简单，但是我开始没有找到`WebView userAgent` 其实发现他不需要。

下面来讲下如何让服务器可以识别访问的是手机。

我们在前台做一个简单页面，开始是一个 WebView 和一个按钮，点击按钮可以获取到手机访问的页面

```xml
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <WebView x:Name="Webview"/>
        <Button Content="手机" Click="WebPhone_OnClick"/>
    </Grid>
```

然后在后台很简单，因为我知道csdn手机返回的和电脑不同于是就使用csdn来访问，看他返回的是不是手机页面。

因为我们需要使用`httpRequestMessage`，他可以有`httpRequestMessage.Headers.Add("User-Agent", userAgent);`添加访问的是手机。那么我们发现`Webview.NavigateWithHttpRequestMessage`可以使用`httpRequestMessage` 于是我把这些写在点击，从点击获取到url的的访问是手机。

```csharp
        private void WebPhone_OnClick(object sender, RoutedEventArgs e)
        {
            var httpRequestMessage = new Windows.Web.Http.HttpRequestMessage(Windows.Web.Http.HttpMethod.Get, new Uri(Url));
            var userAgent = "Mozilla/5.0 (Windows Phone 10.0; Android 4.2.1; WebView/3.0; Microsoft; Virtual) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.135 Mobile Safari/537.36 Edge/12.10240 sample/1.0";
            httpRequestMessage.Headers.Add("User-Agent", userAgent);
            Webview.NavigateWithHttpRequestMessage(httpRequestMessage);
        }
```

这里的`userAgent`可以修改很多其它的浏览器

```csharp
 var userAgent = "Mozilla/5.0 (Windows Phone 10.0; Android 4.2.1; WebView/3.0; Microsoft; Virtual) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.135 Mobile Safari/537.36 Edge/12.10240 sample/1.0";
```

参见http://outofmemory.cn/code-snippet/1901/mobile-liulanqi-User-Agent-summary 这里收集很多浏览器的标识，如果需要的话，直接复制。


```csharp
 Android N1

Mozilla/5.0 (Linux; U; Android 2.3.7; en-us; Nexus One Build/FRF91) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1

Android QQ浏览器 For android

MQQBrowser/26 Mozilla/5.0 (Linux; U; Android 2.3.7; zh-cn; MB200 Build/GRJ22; CyanogenMod-7) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1

Android UC For android

JUC (Linux; U; 2.3.7; zh-cn; MB200; 320*480) UCWEB7.9.3.103/139/999

备注: 320*480 是设备的分辨率,可以修改.

Android Firefox手机版Fennec

Mozilla/5.0 (Windows NT 6.1; WOW64; rv:7.0a1) Gecko/20110623 Firefox/7.0a1 Fennec/7.0a1
Android Opera Mobile

Opera/9.80 (Android 2.3.4; Linux; Opera Mobi/build-1107180945; U; en-GB) Presto/2.8.149 Version/11.10
Android Pad Moto Xoom

Mozilla/5.0 (Linux; U; Android 3.0; en-us; Xoom Build/HRI39) AppleWebKit/534.13 (KHTML, like Gecko) Version/4.0 Safari/534.13
iPhone3

Mozilla/5.0 (iPhone; U; CPU iPhone OS 3_0 like Mac OS X; en-us) AppleWebKit/420.1 (KHTML, like Gecko) Version/3.0 Mobile/1A542a Safari/419.3
iPhone4

Mozilla/5.0 (iPhone; U; CPU iPhone OS 4_0 like Mac OS X; en-us) AppleWebKit/532.9 (KHTML, like Gecko) Version/4.0.5 Mobile/8A293 Safari/6531.22.7
iPad

Mozilla/5.0 (iPad; U; CPU OS 3_2 like Mac OS X; en-us) AppleWebKit/531.21.10 (KHTML, like Gecko) Version/4.0.4 Mobile/7B334b Safari/531.21.10
BlackBerry

Mozilla/5.0 (BlackBerry; U; BlackBerry 9800; en) AppleWebKit/534.1+ (KHTML, like Gecko) Version/6.0.0.337 Mobile Safari/534.1+
WebOS HP Touchpad

Mozilla/5.0 (hp-tablet; Linux; hpwOS/3.0.0; U; en-US) AppleWebKit/534.6 (KHTML, like Gecko) wOSBrowser/233.70 Safari/534.6 TouchPad/1.0
Nokia N97

Mozilla/5.0 (SymbianOS/9.4; Series60/5.0 NokiaN97-1/20.0.019; Profile/MIDP-2.1 Configuration/CLDC-1.1) AppleWebKit/525 (KHTML, like Gecko) BrowserNG/7.1.18124
Windows Phone Mango

Mozilla/5.0 (compatible; MSIE 9.0; Windows Phone OS 7.5; Trident/5.0; IEMobile/9.0; HTC; Titan)
Windows Phone Mango的User Agent格式为:Mozilla/5.0 (compatible; MSIE 9.0; Windows Phone OS 7.5; Trident/5.0; IEMobile/9.0;厂商;型号[;运营商])
```


所有代码

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.InteropServices.WindowsRuntime;
using Windows.Foundation;
using Windows.Foundation.Collections;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Controls.Primitives;
using Windows.UI.Xaml.Data;
using Windows.UI.Xaml.Input;
using Windows.UI.Xaml.Media;
using Windows.UI.Xaml.Navigation;

//“空白页”项模板在 http://go.microsoft.com/fwlink/?LinkId=402352&clcid=0x409 上有介绍

namespace WebViewUwp
{
    /// <summary>
    /// 可用于自身或导航至 Frame 内部的空白页。
    /// </summary>
    public sealed partial class MainPage : Page
    {
        public MainPage()
        {
            this.InitializeComponent();
            Webview.Navigate(new Uri(Url));
        }
        private string Url { set; get; }
        = "http://blog.csdn.net/lindexi_gd";

        private void WebPhone_OnClick(object sender, RoutedEventArgs e)
        {
            var httpRequestMessage = new Windows.Web.Http.HttpRequestMessage(Windows.Web.Http.HttpMethod.Get, new Uri(Url));
            var userAgent = "Mozilla/5.0 (Windows Phone 10.0; Android 4.2.1; WebView/3.0; Microsoft; Virtual) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.135 Mobile Safari/537.36 Edge/12.10240 sample/1.0";
            httpRequestMessage.Headers.Add("User-Agent", userAgent);
            Webview.NavigateWithHttpRequestMessage(httpRequestMessage);
        }
    }
}

```

对于User设置可以参见：http://blog.csdn.net/adc_god/article/details/51951514 ，这位大神收集了很多浏览器


```csharp
    Edge F12
iphone 6
Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13B143 Safari/601.1


Lumia 950
Mozilla/5.0 (Windows Phone 10.0; Android 4.2.1; Microsoft; Lumia 950) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2486.0 Mobile Safari/537.36 Edge/14.14263


Surface Book
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2486.0 Safari/537.36 Edge/13.10586


电脑版本

edge

Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.82 Safari/537.36 Edge/14.14366 


360急速浏览器

Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.101 Safari/537.36


手机版本

华为qq手机浏览器

Mozilla/5.0 (Linux; U; Android 5.1.1; zh-cn; HUAWEI P7-L07 Build/HuaweiP7-L07) AppleWebKit/537.36 (KHTML, like Gecko)Version/4.0 Chrome/37.0.0.0 MQQBrowser/6.8 Mobile Safari/537.36
```


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

