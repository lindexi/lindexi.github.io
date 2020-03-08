# win10 uwp httpClient 登陆CSDN

本文告诉大家如何模拟登陆csdn，这个方法可以用于模拟登陆其他网站。
<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->


<div id="toc"></div>

## HttpClient 使用 Cookie

我们可以使用下面代码让 HttpClient 使用 Cookie ，有了这个才可以保存登陆，不然登陆成功下次访问网页还是没登陆。


```csharp
            CookieContainer cookies = new CookieContainer();

            HttpClientHandler handler = new HttpClientHandler();
            handler.CookieContainer = cookies;
            HttpClient http = new HttpClient(handler);
```

虽然已经有`Cookie`，但是还缺少一些请求需要带的头，因为浏览器是会告诉网站，需要的`Accept`，为了假装这是一个浏览器，所以就需要添加`Accept` 和`Accept-Encoding` `Accept-Language` `User-Agent`

## 添加 Accept

下面的代码可以添加`Accept`，这里后面的字符串可以自己使用浏览器查看，复制。

```csharp
            http.DefaultRequestHeaders.TryAddWithoutValidation("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8");
```

## 添加 Accept-Encoding

```csharp
            http.DefaultRequestHeaders.TryAddWithoutValidation("Accept-Encoding", "gzip, deflate, br");

```

如果有 `gzip` 就需要解压，这个现在不太好弄，建议不要加。

## 添加 Accept-Language

```csharp
            http.DefaultRequestHeaders.TryAddWithoutValidation("Accept-Language", "zh-CN,zh;q=0.8");

```

## 添加 User-Agent

```csharp
http.DefaultRequestHeaders.TryAddWithoutValidation("User-Agent", "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36");
```

更多`User-Agent`请看[win10 uwp 如何让WebView标识win10手机](https://lindexi.oschina.io/lindexi/post/win10-uwp-%E5%A6%82%E4%BD%95%E8%AE%A9WebView%E6%A0%87%E8%AF%86win10%E6%89%8B%E6%9C%BA.html )

## ContentType

如果设置 ContentType 需要在发送的内容进行添加

```csharp
            content = new StringContent("{\"loginName\":\"lindexi\",\"password\":\"csdn\",\"autoLogin\":false}")
            {
                Headers = { ContentType = new MediaTypeHeaderValue("application/json") }
            };
```

## 发送数据

如果需要使用 Post 或 get 发送数据，那么可以使用`HttpContent`做出数据，提供的类型有`StringContent`、`FormUrlEncodedContent`等。

其中`StringContent`最简单，而`FormUrlEncodedContent`可以自动转换。

```csharp
            str = $"username={account.UserName}&password={account.Key}&lt={lt}&execution={execution}&_eventId=submit";
            str = str.Replace("@", "%40");

            HttpContent content = new StringContent(str, Encoding.UTF8);
```

上面代码就是使用 `StringContent` 可以看到需要自己转换特殊字符，当然一个好的方法是使用 urlencoding 转换。

如果使用`FormUrlEncodedContent`就不需要做转换

```csharp
          content=new FormUrlEncodedContent(new List<KeyValuePair<string, string>>()
            {
                new KeyValuePair<string, string>("username",account.UserName),
                new KeyValuePair<string, string>("password",account.Key),
                new KeyValuePair<string, string>("lt",lt),
                new KeyValuePair<string, string>("execution",execution),
                new KeyValuePair<string, string>("_eventId","submit")
            });
```

如果需要上传文件，那么需要使用`MultipartFormDataContent`

```csharp
            content = new MultipartFormDataContent();
            ((MultipartFormDataContent)content).Headers.Add("name", "file1");
           
            ((MultipartFormDataContent)content).Headers.Add("filename", "20170114120751.png");
            var stream = new StreamContent(await File.OpenStreamForReadAsync());
            ((MultipartFormDataContent)content).Add(stream);
```

## 登陆方法

打开 https://passport.csdn.net/account/login 可以看到这个界面

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171027142056.jpg)

右击查看源代码，可以拿到上传需要使用的两个变量 lt 和 execution

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171027142158.jpg)

在登陆的时候，使用 post 把账号密码、lt execution 上传就可以登陆

## 模拟登陆csdn

于是下面就是模拟登陆

1. 获得账号信息

            AccountCimage account = AppId.AccoutCimage;


2. cookie

            CookieContainer cookies = new CookieContainer();

            HttpClientHandler handler = new HttpClientHandler();
            handler.CookieContainer = cookies;
            HttpClient http = new HttpClient(handler);

3. 获得登陆需要的流水号

            var url = new Uri("https://passport.csdn.net/account/login");
   
            http.DefaultRequestHeaders.TryAddWithoutValidation("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8");
            //http.DefaultRequestHeaders.TryAddWithoutValidation("Accept-Encoding", "gzip, deflate, br");
            http.DefaultRequestHeaders.TryAddWithoutValidation("Accept-Language", "zh-CN,zh;q=0.8");
            http.DefaultRequestHeaders.TryAddWithoutValidation("User-Agent", "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36");


            handler.UseCookies = true;
            handler.AllowAutoRedirect = true;

            string str = await http.GetStringAsync(url);
            Regex regex = new Regex(" type=\"hidden\" name=\"lt\" value=\"([\\w|\\-]+)\"");
            var lt = regex.Match(str).Groups[1].Value;
            regex = new Regex("type=\"hidden\" name=\"execution\" value=\"(\\w+)\"");
            var execution = regex.Match(str).Groups[1].Value;

4. 登陆


            str = $"username={account.UserName}&password={account.Key}&lt={lt}&execution={execution}&_eventId=submit";
            str = str.Replace("@", "%40");

            HttpContent content = new StringContent(str, Encoding.UTF8);

      
            str = await content.ReadAsStringAsync();
            content=new FormUrlEncodedContent(new List<KeyValuePair<string, string>>()
            {
                new KeyValuePair<string, string>("username",account.UserName),//.Replace("@", "%40")),
                new KeyValuePair<string, string>("password",account.Key),
                new KeyValuePair<string, string>("lt",lt),
                new KeyValuePair<string, string>("execution",execution),
                new KeyValuePair<string, string>("_eventId","submit")
            });
            str = await content.ReadAsStringAsync();

            str = await (await http.PostAsync(url, content)).Content.ReadAsStringAsync();


5. 查看登陆


   url = new Uri("http://write.blog.csdn.net/");
            str = await http.GetStringAsync(url);


6. 上传文件

           content = new MultipartFormDataContent();
            ((MultipartFormDataContent)content).Headers.Add("name", "file1");
           
            ((MultipartFormDataContent)content).Headers.Add("filename", "20170114120751.png");
            var stream = new StreamContent(await File.OpenStreamForReadAsync());
            ((MultipartFormDataContent)content).Add(stream);
            str = await ((MultipartFormDataContent)content).ReadAsStringAsync();
            url = new Uri("http://write.blog.csdn.net/article/UploadImgMarkdown?parenthost=write.blog.csdn.net");
            var message = await http.PostAsync(url, content);
            if (message.StatusCode == HttpStatusCode.OK)
            {
                ResponseImage(message);
            }

         private async void ResponseImage(HttpResponseMessage message)
         {
            using (MemoryStream memoryStream = new MemoryStream())
            {
                int length = 1024;
                byte[] buffer = new byte[length];
                using (GZipStream zip = new GZipStream(await message.Content.ReadAsStreamAsync(), CompressionLevel.Optimal))
                {
                    int n;
                    while ((n = zip.Read(buffer, 0, length)) > 0)
                    {
                       memoryStream.Write(buffer, 0, n);
                    }
                }

                using (StreamReader stream = new StreamReader(memoryStream))
                {
                    string str = stream.ReadToEnd();
                }
            }
        }


### 使用 WebView 模拟登陆 csdn

下面给大家一个叫简单方法模拟登陆csdn

```csharp
          GeekWebView.Navigate(new Uri("http://passport.csdn.net/"));

            GeekWebView.NavigationCompleted += OnNavigationCompleted;


            F = async () =>
            {

                var functionString = string.Format(@"document.getElementsByName('username')[0].value='{0}';", "lindexi_gd@163.com");
                await GeekWebView.InvokeScriptAsync("eval", new string[] { functionString });
                functionString = string.Format(@"document.getElementsByName('password')[0].value='{0}';", "密码");
                await GeekWebView.InvokeScriptAsync("eval", new string[] { functionString });

                functionString = string.Format(@"document.getElementsByClassName('logging')[0].click();");
                await GeekWebView.InvokeScriptAsync("eval", new string[] { functionString });
            };

        private Action F { set; get; }

        private void OnNavigationCompleted(WebView sender, WebViewNavigationCompletedEventArgs args)
        {
            F();
        }
```

当然，这时需要修改登陆信息，我上面写的是我的。如果遇到有验证码，那么这个方法是不可使用，因为输入验证码暂时还没法做。




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 