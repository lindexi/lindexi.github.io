
# win10 UWP 访问网页

本文告诉大家如何使用多个方式访问网页，可以获得网页源代码，可以做爬取网络信息。

<!--more-->



<div id="toc"></div>
<!-- csdn -->

Windows10 UWP 要访问 csdn博客，可以使用`Windows.Web.Http.HttpClient`，下面尝试访问一下我的bo

```C#

            string str = "http://blog.csdn.net/lindexi_gd/article/details/50392343";

            using (Windows.Web.[Http.HttpClient](Http.HttpClient ) client = new Windows.Web.[Http.HttpClient())](Http.HttpClient()) )

            {

                try

                {

                    Windows.Web.Http.HttpResponseMessage response = await client.GetAsync(new Uri(str));

                    if (response != null && response.StatusCode == Windows.Web.[Http.HttpStatusCode.Ok)](Http.HttpStatusCode.Ok) )

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

在前台有一个TextBlock，名字 tb 和 按钮,按钮点击触发上面代码，访问博客，得到的内容放在 tb

界面看起就是

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F201732119010.jpg)


除了 httpClient 还可以使用 `HttpWebRequest` ，请看下面

```C#

            System.Net.HttpWebRequest request = null;

            request = System.Net.WebRequest.Create(str) as System.Net.[HttpWebRequest;](HttpWebRequest; )

            request.Accept = "text/html, application/xhtml+xml, image/jxr, */*";
            //有些网站需要 Accept 如果这个不对，不返回

            request.Method = "GET";

            request.CookieContainer = new System.Net.CookieContainer();

            try

            {

                System.Net.HttpWebResponse response = (System.Net.HttpWebResponse)await request.GetResponseAsync();

                if (response != null && response.StatusCode==System.Net.[HttpStatusCode.OK)](HttpStatusCode.OK) )

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

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F201732119047.jpg)

参见：
[win10 uwp 模拟网页输入](https://lindexi.oschina.io/lindexi/post/win10-uwp-%E6%A8%A1%E6%8B%9F%E7%BD%91%E9%A1%B5%E8%BE%93%E5%85%A5.html )

[如何使用 C# 爬虫获得专栏博客更新排行](http://lindexi.oschina.io/lindexi//post/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-C-%E7%88%AC%E8%99%AB%E8%8E%B7%E5%BE%97%E4%B8%93%E6%A0%8F%E5%8D%9A%E5%AE%A2%E6%9B%B4%E6%96%B0%E6%8E%92%E8%A1%8C/ )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。