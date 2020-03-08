# win10 uwp 客户端如何发送类到 asp dotnet core 作为参数

本文告诉大家如何在 UWP 或 WPF 客户端通过将类转换为 json 发送到 asp dotnet core 作为方法的参数

<!--more-->
<!-- CreateTime:2019/2/19 9:14:09 -->

<!-- csdn -->

熟悉客户端开发的小伙伴在看到 asp dotnet core 会发现在使用的时候实际上只是在方法上面添加一个特性，这时通过 URL 访问的数据就会通过路由调用到这个方法，那么如何在客户端通过 URL 调用到方法在方法里面传入类参数？

假设有这样的一个类

```csharp
    public class KebunerNeefunadrow
    {
        /// <summary>
        /// 包的名
        /// </summary>
        public string Name { get; set; }

        public string Version { set; get; }
    }
```

需要将这个类作为参数，发送到 asp dotnet core 的一个方法里面

```csharp
        [HttpPost("Download")]
        public ActionResult Download([FromBody]KebunerNeefunadrow saljudecooBolor)
```

可以通过在客户端用 Json.NET 将类转换为 json 然后调用 URL 这样默认通过 json 转换的 asp dotnet core 就会将 json 转换为类，然后传入下载方法

这里的 asp dotnet core 的方法有两个要求，第一个是特性使用 Post 方法 `[HttpPost("Download")]` 这样就告诉 asp dotnet core 这个方法需要通过 Post 调用，同时这个方法的 URL 是 `api/xx/Download` 传入的参数同样有一个特性 `[FromBody]` 这个特性告诉 asp dotnet core 这个参数从 Post 的内容拿到

在客户端需要通过下面代码转换类为json才可以发送

```csharp
            var kebunerNeefunadrow = new KebunerNeefunadrow()
            {
                Name = "lindexi",
                Version = new Version("5.1.2").ToString()
            };

            var json = JsonConvert.SerializeObject(kebunerNeefunadrow);

```

发送 Json 的方法可以通过 HttpClient 发送内容，发送的内容需要通过 ContentType 告诉服务器发送的是 json 才可以

```csharp
            var stringContent = new StringContent(json);
            stringContent.Headers.ContentType.MediaType = "application/json";
```

特别注意，默认发送的是文本，需要通过 ContentType 修改为 json 才可以在服务器使用 json转换

通过 httpClient 发送的代码请看下面

```csharp
            var json = JsonConvert.SerializeObject(kebunerNeefunadrow);

            var response = await httpClient.PostAsync(url, stringContent);
```

这样就可以在 UWP 或 WPF 客户端调用 asp dotnet core 的方法，在这个方法传入参数

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
