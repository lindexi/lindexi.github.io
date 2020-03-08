# WPF 获取下载内容长度

本文告诉大家如何获取一个下载内容的长度

<!--more-->
<!-- CreateTime:2018/11/8 20:18:15 -->

<!-- csdn -->

在 WPF 可以通过 System.Net 的类进行下载资源，如下载一张图片 [http://image.acmx.xyz/lindexi%2F2018116203842298](http://image.acmx.xyz/lindexi%2F2018116203842298) 可以使用下面的代码，通过 ContentLength 拿到下载的内容长度

```csharp
        private async void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var url = "http://image.acmx.xyz/lindexi%2F2018116203842298";
            var request = WebRequest.CreateHttp(url);
            request.UserAgent = "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.1";
            request.Method = "Get";
            using (var response = await request.GetResponseAsync())
            {
                var length = response.ContentLength;
            }
        }
```
这里的 ContentLength 就是内容的长度，注意很多的网站都需要使用 UserAgent 可以从 [UserAgentString](http://www.useragentstring.com/ ) 找到很多可以使用的资源

实际上面的代码可以使用 HEAD 代替 get 方法，请看下面

```csharp
            var url = "http://image.acmx.xyz/lindexi%2F2018116203842298";
            var request = WebRequest.CreateHttp(url);
            request.UserAgent = "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.1";
            request.Method = "HEAD";
            using (var response = await request.GetResponseAsync())
            {
                var length = response.ContentLength;
            }
```

具体请看 [HTTP协议中POST、GET、HEAD、PUT等请求方法及相应值得含义 - 空白_回忆的博客 - CSDN博客](https://blog.csdn.net/qq_26291823/article/details/51900422 )

[c# - Getting correct download file size from url - Stack Overflow](https://stackoverflow.com/a/52028622/6116637 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
