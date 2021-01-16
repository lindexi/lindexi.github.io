
# C# dotnet 编写 curl 的用户名和密码逻辑

在使用 curl 时，可以采用 -u 加上用户名和密码，这个对应在 C# 里面也是两句话的事情。只是在 curl 里面封装好了，我通过工具人协助解决，于是就在这里记录一下

<!--more-->


<!-- 发布 -->

假定使用 curl 输入的是如下代码

```
curl -ulindexi:FD7doYUzM7WApXobRb7X9qgURCF -T "E:\lindexi\doubi.exe" "https://blog.lindexi.com/artifactory/doubi.exe"
```

通过上面代码可以给我的存储服务上传文件，而如果使用 dotnet 编写，其实逻辑也十分简单

在账号密码这里，假定有变量 userName 和 key 分别是账号和密码，那么构建上传文件使用的账号密码将放在 Http 的 Header 里面，代码如下

```csharp
            var encodedAuthString = Convert.ToBase64String(Encoding.ASCII.GetBytes($"{userName}:{key}"));

            httpClient.DefaultRequestHeaders.Add("Authorization", $"Basic {encodedAuthString}");
```

而上传文件调用的是 Put 接口，整体逻辑如下

```csharp
            var url = "https://blog.lindexi.com/artifactory/doubi.exeg";
            var file = @"E:\lindexi\doubi.exe";

            var userName = "lindexi";
            var key = "FD7doYUzM7WApXobRb7X9qgURCF";

            var encodedAuthString = Convert.ToBase64String(Encoding.ASCII.GetBytes($"{userName}:{key}"));

            var httpClient = new HttpClient();
            httpClient.DefaultRequestHeaders.Add("Authorization", $"Basic {encodedAuthString}");

            await httpClient.PutAsync(url, new StreamContent(File.OpenRead(file)));
```

看起来代码还是十分简单的，于是我就使用这个方法，在 GitHub 的 Action 上帮忙下载外网的内容，然后发送到我的国内服务器上

感谢工具人 [lsj](https://blog.sdlsj.net/ ) 提供的方法





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。