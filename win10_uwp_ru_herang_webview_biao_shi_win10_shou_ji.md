# win10 uwp 如何让WebView标识win10手机

本文主要：如何让WebView访问的网页设别为手机，当然这句话我说不好，换个，如何让WebView设别为手机。上面两句话都是错的，因为是服务器识别，不是网页，第二句话应该是让服务器而不是WebView。为什么这样写是因为有大神在群里问这个，他这样说，我这样写希望大家能在搜索看到。当然本文发在csdn和win10.me，在他地方并没有发，不过我的gitbook.io还是有的。

如何让WebView设别手机，其实很简单，但是我开始没有找到WebView userAgent 其实发现他不需要。

我们在前台

```
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <WebView x:Name="Webview"/>
        <Button Content="手机" Click="WebPhone_OnClick"/>
    </Grid>
```

然后在后台很简单

因为我们需要使用httpRequestMessage，他可以有`httpRequestMessage.Headers.Add("User-Agent", userAgent);`那么我们发现Webview.NavigateWithHttpRequestMessage

我把这些写按钮	

```
        private void WebPhone_OnClick(object sender, RoutedEventArgs e)
        {
            var httpRequestMessage = new Windows.Web.Http.HttpRequestMessage(Windows.Web.Http.HttpMethod.Get, new Uri(Url));
            var userAgent = "Mozilla/5.0 (Windows Phone 10.0; Android 4.2.1; WebView/3.0; Microsoft; Virtual) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.135 Mobile Safari/537.36 Edge/12.10240 sample/1.0";
            httpRequestMessage.Headers.Add("User-Agent", userAgent);
            Webview.NavigateWithHttpRequestMessage(httpRequestMessage);
        }
```

这里

```
 var userAgent = "Mozilla/5.0 (Windows Phone 10.0; Android 4.2.1; WebView/3.0; Microsoft; Virtual) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.135 Mobile Safari/537.36 Edge/12.10240 sample/1.0";
```

可以修改，参见http://outofmemory.cn/code-snippet/1901/mobile-liulanqi-User-Agent-summary

所有代码

```
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

对于User设置可以参见：http://blog.csdn.net/adc_god/article/details/51951514

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。