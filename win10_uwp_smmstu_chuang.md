# win10 uwp smms图床

本文，如何使用smms图床上传图片，用到win10 uwp post文件，因为我是渣渣，如果本文有错的，请和我说，在本文评论，或发给我邮箱，请不要发不良言论
<!--more-->

<div id="toc"></div>

找到一个很好的图床，sm.ms

可以简单使用post上传文件，我就做了一个工具，可以把图片上传，使用只需要

```csharp
            //传入文件
            smms.Model.Imageshack imageshack = new Imageshack()
            {
                File=File,
            };
            //上传完成事件，其中str为sm.ms返回，一般为json
            //Reminder是例子，可以根据具体修改，注意要同步CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync
            imageshack.OnUploadedEventHandler += (sender, str) => Reminder = str.Replace("\\/","/");
            //上传
            imageshack.UpLoad();
```

我将会把我做的发现的和大家说

## 进行HttpClient post参数错误

从“Windows.Web.Http.HttpStringContent”转换为“System.Net.Http.HttpContent”

原因

用了`System.Net.Http.HttpClient`其实HttpStringContent是可以在错误看到，不是System.Net.Http

方法

使用

```csharp
           Windows.Web.Http.HttpClient webHttpClient=
                new Windows.Web.Http.HttpClient();

           Windows.Web.Http.HttpStringContent httpString=
                new HttpStringContent("http://blog.csdn.net/lindexi_gd");
            await webHttpClient.PostAsync(new Uri(url), httpString);
```


## win10 uwp post 上传文件

我们可以使用HttpMultipartFormDataContent上传
其中我们需要从文件转流，打开StorageFile，把它转换HttpStreamContent

            var fileContent = new HttpStreamContent(await File.OpenAsync(FileAccessMode.Read));

然后我们要fileContent.Headers.Add("Content-Type", "application/octet-stream");

我们可以把httpMultipartFormDataContent加上fileContent，看到sm.ms

|参数名称|类型|是否必须|描述|
|--|--|--|--|
|smfile|File|是|表单名称。上传图片用到|
|ssl	|Bool|	否|	是否使用 https 输出，默认关闭|
|format	|String|	否|	输出的格式。可选值有 json、xml。默认为 json|
|domain|	Int|	否|	图片域名。可选|

我们就修改`Add(IHttpContent content, System.String name, System.String fileName);` name "smfile"

    httpMultipartFormDataContent.Add(fileContent, "smfile", File.Name);

使用`await webHttpClient.PostAsync(new Uri(url), httpMultipartFormDataContent);`

因为需要拿到上传图片

```csharp
var str = await webHttpClient.PostAsync(new Uri(url), httpMultipartFormDataContent);
            ResponseString = str.Content.ToString();
            OnUploadedEventHandler?.Invoke(this,ResponseString);
```

## 所有代码

https://github.com/lindexi/Imageshack/tree/master/smms

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。




