# win10 uwp HttpClient post错误


进行HttpClient post参数错误

从“Windows.Web.Http.HttpStringContent”转换为“System.Net.Http.HttpContent”
<!--more-->

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





