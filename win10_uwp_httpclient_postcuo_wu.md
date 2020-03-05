# win10 uwp HttpClient post错误


进行HttpClient post参数错误

从“Windows.Web.Http.HttpStringContent”转换为“System.Net.Http.HttpContent”
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>
<!-- csdn -->

原因

用了`System.Net.Http.HttpClient`其实HttpStringContent是可以在错误看到，不是System.Net.Http

方法

使用

```csharp
           Windows.Web.Http.HttpClient webHttpClient=
                new Windows.Web.Http.HttpClient();

           Windows.Web.Http.HttpStringContent httpString=
                new HttpStringContent("a");
            await webHttpClient.PostAsync(new Uri(url), httpString);
```


<a href="https://www.codeproject.com/script/Articles/BlogFeedList.aspx?amid=12520573" rel="tag">CodeProject</a>

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。