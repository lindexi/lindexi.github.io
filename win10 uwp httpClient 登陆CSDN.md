# win10 uwp httpClient 登陆CSDN


<!--more-->

## HttpClient 使用 Cookie

我们可以使用下面代码让 HttpClient 使用 Cookie


```csharp
            CookieContainer cookies = new CookieContainer();

            HttpClientHandler handler = new HttpClientHandler();
            handler.CookieContainer = cookies;
            HttpClient http = new HttpClient(handler);
```



```csharp
            AccountCimage account = AppId.AccoutCimage;
            //https://stackoverflow.com/questions/41599384/httpclient-cookie-issue
            CookieContainer cookies = new CookieContainer();

            HttpClientHandler handler = new HttpClientHandler();
            handler.CookieContainer = cookies;
            HttpClient http = new HttpClient(handler);
            var url = new Uri("https://passport.csdn.net/account/login");
            //https://passport.csdn.net/?service=http://write.blog.csdn.net/
            //HttpRequestHeader.
            //http.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8"));
            //http.DefaultRequestHeaders.AcceptEncoding.Add(StringWithQualityHeaderValue.Parse("gzip, deflate, br"));
            //http.DefaultRequestHeaders.UserAgent.Add(new ProductInfoHeaderValue("Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36"));

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

            str = $"username={account.UserName}&password={account.Key}&lt={lt}&execution={execution}&_eventId=submit";
            str = str.Replace("@", "%40");

            HttpContent content = new StringContent(str, Encoding.UTF8);

            //((StringContent)content).Headers
            //username=lindexi_gd%40163.com&password=Huc_3113006277&lt=LT-541277-RgRpxUYUjiMNRYpXclGBNBy0pInajL&execution=e10s1&_eventId=submit
            //username=lindexi_gd%40163.com&password=Huc_3113006277&lt=LT-541546-HzhKAKoaeftqtL6EtiCQYXrC3d716w&execution=e1s1&_eventId=submit
            str = await content.ReadAsStringAsync();
            //content=new
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

            //str = await http.GetStringAsync(url);
            //foreach (Cookie temp in handler.CookieContainer.GetCookies(url))
            //{
            //    handler.CookieContainer.SetCookies(new Uri("http://write.blog.csdn.net/"),temp.ToString());
            //}

            //var temp = handler.CookieContainer.GetCookies(url);

            //handler.CookieContainer.GetCookies()
            url = new Uri("http://write.blog.csdn.net/");
            str = await http.GetStringAsync(url);

            //content=new MultipartContent();
            //((MultipartContent)content)
            content = new MultipartFormDataContent();
            ((MultipartFormDataContent)content).Headers.Add("name", "file1");
            //((MultipartFormDataContent)content)
            ((MultipartFormDataContent)content).Headers.Add("filename", "20170114120751.png");
            var stream = new StreamContent(await File.OpenStreamForReadAsync());
            ((MultipartFormDataContent)content).Add(stream);
            str = await ((MultipartFormDataContent)content).ReadAsStringAsync();
            url = new Uri("http://write.blog.csdn.net/article/UploadImgMarkdown?parenthost=write.blog.csdn.net");
            var message = await http.PostAsync(url, content);
            if (message.StatusCode == HttpStatusCode.OK)
            {
                //str = await message.Content.ReadAsStringAsync();
                //message.Content.ReadAsStreamAsync()
                ResponseImage(message);
            }
```


```csharp
        private async void ResponseImage(HttpResponseMessage message)
        {
            //using (MemoryStream memoryStream = new MemoryStream())
            //{
            //    int length = 1024;
            //    byte[] buffer = new byte[length];
            //    using (GZipStream zip = new GZipStream(await message.Content.ReadAsStreamAsync(), CompressionLevel.Optimal))
            //    {
            //        int n;
            //        while ((n = zip.Read(buffer, 0, length)) > 0)
            //        {
            //            memoryStream.Write(buffer, 0, n);
            //        }
            //    }

            //    using (StreamReader stream = new StreamReader(memoryStream))
            //    {
            //        string str = stream.ReadToEnd();
            //    }
            //}

            using (StreamReader stream = new StreamReader(await message.Content.ReadAsStreamAsync()))
            {
                string str = stream.ReadToEnd();
            }
            //System.IO.Compression.DeflateStream

        }
```
