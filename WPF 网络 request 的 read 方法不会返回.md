# WPF 网络 request 的 read 方法不会返回

我最近为宝资通写软件，这个软件需要从网上下网页。但是使用 HttpRequest 的时候发现 StreamReader 等方法可能等待很久不会返回，能否有方法设置超时解决卡住线程

<!--more-->
<!-- CreateTime:2019/6/23 11:26:26 -->

<!-- csdn -->

使用 HttpRequest 的时候，用 Stream 的方法读取需要服务器返回，如果服务器没有返回，那么这个同步方法将会卡住不会返回

可以通过 ReadWriteTimeout 设置超时时间

```
request.ReadWriteTimeout = 20000;
```


加上超时，就让应用不会在服务器没有返回等待太久。因为有些时候服务器或网络差，不能在比较短时间返回，我们设置这个不能太小。

```
               HttpWebRequest request = (HttpWebRequest) WebRequest.Create(url);
                request.Timeout = 20000;
                //网络响应
                request.ReadWriteTimeout = 20000;

                //request.Headers.Add(HttpRequestHeader.Accept, "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8");
                //request.Headers["Accept"] = "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8";
                request.Method = "GET";
                request.Accept = "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8";
                //request.CachePolicy=new HttpRequestCachePolicy()
                //{

                //};

                request.ContentType = "application/x-www-form-urlencoded";
                request.UserAgent =
                    "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.101 Safari/537.36";
                request.AllowAutoRedirect = true;
                HttpWebResponse response = (HttpWebResponse) request.GetResponse();
                if (response.StatusCode == HttpStatusCode.OK)
                {
                    using (StreamReader stream = new StreamReader(response.GetResponseStream()))
                    {
                        return stream.ReadToEnd().Replace("\\u0027", "'");
                    }
                }
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
