# win10 UWP 访问网页

本文告诉大家如何使用多个方式访问网页，可以获得网页源代码，可以做爬取网络信息。

<!--more-->

<div id="toc"></div>
<!-- csdn -->

Windows10 UWP 要访问 csdn博客，可以使用`Windows.Web.Http.HttpClient`，下面尝试访问一下我的博客 <http://blog.csdn.net/lindexi_gd/article/details/50392343>

我先在 xaml 添加一个 TextBlock ，这个 TextBlock 是 `tb` 用来拿到我访问页面拿到的内容

```C#

            string str = "http://blog.csdn.net/lindexi_gd/article/details/50392343";

            using (Windows.Web.Http.HttpClient client = new Windows.Web.Http.HttpClient())

            {

                try

                {

                    Windows.Web.Http.HttpResponseMessage response = await client.GetAsync(new Uri(str));

                    if (response != null && response.StatusCode == Windows.Web.Http.HttpStatusCode.Ok)

                    {

                        using (Windows.Storage.Streams.InMemoryRandomAccessStream stream = new Windows.Storage.Streams.InMemoryRandomAccessStream())

                        {

                            await response.Content.WriteToStreamAsync(stream);

                            stream.Seek(0);                            

                            Windows.Storage.Streams.Buffer buffer = new Windows.Storage.Streams.Buffer((uint)stream.Size);

                            await stream.ReadAsync(buffer, (uint)stream.Size, Windows.Storage.Streams.InputStreamOptions.Partial);

                            using (Windows.Storage.Streams.DataReader dataReader = Windows.Storage.Streams.DataReader.FromBuffer(buffer))

                            {

                                tb.Text = dataReader.ReadString((uint)stream.Size);

                            }

                        }

                    }

                }

                catch

                {

                }

```

在前台有一个TextBlock，名字是 tb ，界面还有一个 按钮，按钮点击触发上面代码，访问博客，得到的内容放在 tb 显示

这时按下 F5 运行，可以看到下面的界面

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F201732119010.jpg)


除了 httpClient 还可以使用 `HttpWebRequest` ，请看下面

```C#

            System.Net.HttpWebRequest request = null;

            request = System.Net.WebRequest.Create(str) as System.Net.HttpWebRequest;

            request.Accept = "text/html, application/xhtml+xml, image/jxr, */*";
            //有些网站需要 Accept 如果这个不对，不返回

            request.Method = "GET";

            request.CookieContainer = new System.Net.CookieContainer();

            try

            {

                System.Net.HttpWebResponse response = (System.Net.HttpWebResponse)await request.GetResponseAsync();

                if (response != null && response.StatusCode==System.Net.HttpStatusCode.OK)

                {

                    tb.Text = response.ContentLength.ToString();

                    using (Stream stream= response.GetResponseStream())

                    {

                        byte[] buffer = new byte[10240];
                        //实际可以用其他方法

                        stream.Read(buffer, 0, 10240);

                        tb.Text = System.Text.Encoding.UTF8.GetString(buffer);
                        //在哪知道是UTF8？实际上解析网页这里比较难，我用的是知道他是 UTF8

                    }

                }

            }

            catch

            {

            }

```

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F201732119047.jpg)

需要注意 `Windows.Web.Http.HttpClient` 和 `System.Net.Http.HttpClient` 是不相同，请看[揭秘Windows10 UWP中的httpclient接口[2] - 蘑菇先生 - 博客园](https://www.cnblogs.com/mushroom/p/5079964.html ) 和 void 大神写的 [详解 UWP (通用 Windows 平台) 中的两种 HttpClient API](https://validvoid.net/demystifying-httpclient-apis-in-the-uwp/ )

## 设置代理

现在的 UWP 程序只能使用 IE 的代理，而不能自定义代理，虽然存在 httpClientHandler.Proxy 可以设置 IWebProxy ，我也尝试写了自己的本地代理，但是没有访问

```csharp
    public class WebProxy : IWebProxy
    {
        /// <inheritdoc />
        public Uri GetProxy(Uri destination)
        {
            return new Uri("socks5://127.0.0.1:10112");
        }

        /// <inheritdoc />
        public bool IsBypassed(Uri host)
        {
            return false;
        }

        /// <inheritdoc />
        public ICredentials Credentials { get; set; }
    }
```

我在 GetProxy 使用断点，在使用下面代码运行，没有进入刚才写的函数

```csharp
            var httpClientHandler = new HttpClientHandler();
            httpClientHandler.UseProxy = true;
            httpClientHandler.Proxy = new WebProxy();

            var httpClient = new HttpClient(httpClientHandler);

            var str = await httpClient.GetStringAsync(new Uri("https://www.google.com"));

            Debug.WriteLine(str);
```

## WebView

还有一个简单的方法是使用 WebView 就是 Edge 浏览器，所以通过浏览器可以做出更强大的效果。

先在界面添加一个按钮和控件

```csharp
        <WebView x:Name="TraymorxasluPoocigur"></WebView>
        <Button HorizontalAlignment="Center" Content="确定" Click="FersamaltaiJearxaltray_OnClick"></Button>
```

在按钮点击的时候，尝试下面几个方式访问网页

```csharp
        private void FersamaltaiJearxaltray_OnClick(object sender, RoutedEventArgs e)
        {
            TraymorxasluPoocigur.Navigate(new Uri("http://lindexi.github.io"));
        }
```

访问解决方案资源

```csharp
        private void FersamaltaiJearxaltray_OnClick(object sender, RoutedEventArgs e)
        {
            try
            {
                TraymorxasluPoocigur.Navigate(new Uri("ms-appx:///林德熙.html"));
            }
            catch (Exception exception)
            {
                Debug.WriteLine(exception.Message);
            }
        }
```

参见：[win10 uwp 访问解决方案文件](https://lindexi.gitee.io/post/win10-uwp-%E8%AE%BF%E9%97%AE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%96%87%E4%BB%B6.html )

访问本地的文件

```csharp
                var file = await StorageFile.GetFileFromApplicationUriAsync(new Uri("ms-appx:///林德熙.html"));

                var folder = ApplicationData.Current.LocalFolder;

                var str = await FileIO.ReadTextAsync(file);

                file = await folder.CreateFileAsync("林德熙.html", CreationCollisionOption.ReplaceExisting);

                await FileIO.WriteTextAsync(file, str);

                TraymorxasluPoocigur.Navigate(new Uri("ms-appdata:///local/林德熙.html"));
```

访问字符串

```csharp
                var file = await StorageFile.GetFileFromApplicationUriAsync(new Uri("ms-appx:///林德熙.html"));

                var str = await FileIO.ReadTextAsync(file);

                TraymorxasluPoocigur.NavigateToString(str);
```

参见：
[win10 uwp 模拟网页输入](https://lindexi.oschina.io/lindexi/post/win10-uwp-%E6%A8%A1%E6%8B%9F%E7%BD%91%E9%A1%B5%E8%BE%93%E5%85%A5.html )

[如何使用 C# 爬虫获得专栏博客更新排行 - CSDN博客](https://blog.csdn.net/lindexi_gd/article/details/72516802 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
