# win10 UWP 访问网页

<!--more-->
<!-- csdn -->

Windows10 UWP 要访问博客，可以使用Windows.Web.Http.HttpClient

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

在前台有一个TextBlock，名字tb 和 按钮


按钮点击触发上面代码，访问博客，得到的内容放在tb

界面看起就是

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F201732119010.jpg)


除了 httpClient 还可以使用
HttpWebRequest

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

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F201732119047.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
