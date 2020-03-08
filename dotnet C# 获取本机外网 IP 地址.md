# dotnet C# 获取本机外网 IP 地址

本文通过 SOHU 提供的服务获取本机的外网 IP 地址

<!--more-->
<!-- CreateTime:2019/11/17 16:38:10 -->

<!-- csdn -->

如果有自己的服务器，可以通过自己的服务器使用 [asp dotnet core 服务器获取客户 IP 地址](https://blog.lindexi.com/post/asp-dotnet-core-%E4%BB%8E-Frp-%E8%8E%B7%E5%8F%96%E7%94%A8%E6%88%B7%E7%9C%9F%E5%AE%9E-IP-%E5%9C%B0%E5%9D%80.html ) 方法，将获取的 IP 地址返回给用户

如果没有搭建服务器，可以使用 SOHU 的方法

访问 [http://pv.sohu.com/cityjson](http://pv.sohu.com/cityjson) 可以返回当前设备的外网 IP 地址

所以使用下面代码可以获取

```csharp
            var httpClient = new HttpClient();

            var str = await httpClient.GetStringAsync("http://pv.sohu.com/cityjson");
```

但是 SOHU 返回的使用 GBK 编码，可以通过 [dotnet core 使用 GBK 编码](https://blog.lindexi.com/post/dotnet-core-%E4%BD%BF%E7%94%A8-GBK-%E7%BC%96%E7%A0%81.html ) 的方法，安装 System.Text.Encoding.CodePages 库 然后注册

```csharp
            Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);

            var httpClient = new HttpClient();

            var str = await httpClient.GetStringAsync("http://pv.sohu.com/cityjson");

            Console.WriteLine(str); // var returnCitySN = {"cip": "183.63.127.82", "cid": "440100", "cname": "广东省广州市"};
```

此时返回的字符串不是只有 IP 需要读取字符串

```csharp
            var regex = new Regex(@"(\d+\.\d+\.\d+\.\d+)");
            var match = regex.Match(str);
            if (match.Success)
            {
                var ip = match.Groups[0].Value;
                Console.WriteLine(ip);
            }
```

这样就可以拿到本机的外网 IP 地址

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0de1af7a6591bf88cc901a23a75bbc33a6061413/RernallkarhadahiNearlaynerene) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
