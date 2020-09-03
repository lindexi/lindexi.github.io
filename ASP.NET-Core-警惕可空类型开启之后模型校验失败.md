
# ASP.NET Core 警惕可空类型开启之后模型校验失败

在开启 Nullable 可空类型之后，原本可以调用的 API 也许就会提示 400 BadRequest 因为传入参数不合法，模型校验失败，此时将不会进入预期的 API 函数，同时也不会在输出里面找到有用的信息

<!--more-->


<!-- CreateTime:2020/8/26 14:40:08 -->

<!-- 发布 -->

在 SDK 风格的 csproj 文件开启可空类型可以添加下面代码

```xml
      <Nullable>enable</Nullable>
```

为了方便让小伙伴知道上面代码加在哪里，我贴出更多的 csproj 文件代码

```xml
  <PropertyGroup>
      <TargetFramework>netcoreapp3.1</TargetFramework>
      <Nullable>enable</Nullable>
  </PropertyGroup>
```

在开启之后，原本工作的很好的 API 也许在客户端调用的时候，将会提示 400 BadRequest 内容大概如下

```
{
    "type":"https://tools.ietf.org/html/rfc7231#section-6.5.1",
    "title":"One or more validation errors occurred.",
    "status":400,
    "traceId":"00-99b1c07693a55c4990389901832992a4-b2ed63ee45e85344-01",
    "errors":
    {
        "Account":
        [
            "The Account field is required."
        ]
    }
}
```

复习一下为什么会存在 400 错误

- 也许调用的 API 错误了，本来是预期调用 Foo 的，但却调用了 A 接口
- 也许调用的端口不对，也许是被 Fiddler 干扰了
- 也许是传入的参数不合法

如上面提示，实际内容是 `The Account field is required` 翻译过来就是接口里面的参数，要求一定存在 Account 属性

而明明之前工作的好好的，接口实现如下

```csharp
        [HttpPost]
        // ReSharper disable once StringLiteralTypo
        [Route("/lindexi/doubi")]
        [RequestSizeLimit(100_000_000)]
        public async Task<string> PostFile([FromForm] LindexiUploadFileRequest request)
```

也就是需要通过 FromForm 拿到内容，而 LindexiUploadFileRequest 的定义如下

```csharp
    [DataContract]
    public class LindexiUploadFileRequest
    {
        [DataMember(Name = "file")]
        [JsonPropertyName("file")]
        public IFormFile File { get; set; }

        [DataMember(Name = "account")]
        [JsonPropertyName("account")]
        public string Account { get; set; }
    }
```

客户端调用代码大概如下

```csharp
       public async Task<string> Upload(string host, string file)
        {
            var multipartFormDataContent = new MultipartFormDataContent();
            var fileName = Path.GetFileName(file);

            var stringContent = new StringContent(fileName);
            multipartFormDataContent.Add(stringContent, "Name");

            using var fileStream = new FileStream(file, FileMode.Open);
            using var streamContent = new StreamContent(fileStream);
            multipartFormDataContent.Add(streamContent, "File", fileName);

            var account = "";

            multipartFormDataContent.Add(new StringContent(account), "Account");

            var httpClient = new HttpClient();
            var url = $"{host}/lindexi/doubi";

            var response = await httpClient.PostAsync(url, multipartFormDataContent).ConfigureAwait(false);
            var content = await response.Content.ReadAsStringAsync().ConfigureAwait(false);
            return $"{response.StatusCode}\n{content}";
        }
```

小伙伴是否可以看出问题？实际上在开启可空之后，尽管在客户端代码里面设置了 `multipartFormDataContent.Add(new StringContent(account), "Account");` 但是传入的内容是空字符串

而开启可空之后，定义的数据模型 `public string Account { get; set; }` 表示 Account 一定不是空，于是传入空的 Account 属性将会校验不通过

有两个解决方法，第一个解决方法就是标记 Account 属性可空

```csharp
        [DataMember(Name = "account")]
        [JsonPropertyName("account")]
        public string? Account { get; set; }
```

但是对于大项目，很难测试全，此时可以在全局配置，让行为和之前相同

```csharp
services.AddControllers(options => options.SuppressImplicitRequiredAttributeForNonNullableReferenceTypes = true);
```

打开 Startup.cs 文件，在 ConfigureServices 函数添加上面代码即可

但对于 EF 这边，有更多的变更，详细请看 [Working with nullable reference types - EF Core](https://docs.microsoft.com/en-us/ef/core/miscellaneous/nullable-reference-types )

因此如果是新项目，我推荐开启可空，而对于现有的项目，我不推荐打开

[MvcOptions.SuppressImplicitRequiredAttributeForNonNullableReferenceTypes Property (Microsoft.AspNetCore.Mvc)](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.mvcoptions.suppressimplicitrequiredattributefornonnullablereferencetypes?view=aspnetcore-3.1&WT.mc_id=DX-MVP-5003606 )

[Nullable=Enabled results in required validation errors for parameters or bound properties with default values · Issue #18403 · dotnet/aspnetcore](https://github.com/dotnet/aspnetcore/issues/18403 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。