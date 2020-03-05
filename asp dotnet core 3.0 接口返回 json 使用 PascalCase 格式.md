# asp dotnet core 3.0 接口返回 json 使用 PascalCase 格式

在 asp dotnet core 3.0 默认的 webapi 返回接口都是返回 json 格式，同时这个 json 格式使用的是 CamelCase 属性名风格。如果想要兼容之前的格式，让 webapi 返回的 json 的属性名使用 PascalCase 格式，那么请看本文

<!--more-->
<!-- CreateTime:2019/9/23 18:39:17 -->

<!-- csdn -->

默认的 ASP.NET Core 3.0 的 WebAPI 的 json 返回值的属性使用首字符小写的 CamelCase 属性名风格，可以通过在 ConfigureServices 方法配置让返回值属性使用其他风格

最简单的方法是设置 PropertyNamingPolicy 属性，请看代码

```csharp
            services.AddControllers()
                .AddJsonOptions(options => options.JsonSerializerOptions.PropertyNamingPolicy = null);
```

另一个是通过 NewtonsoftJson 设置

首先安装 [Microsoft.AspNetCore.Mvc.NewtonsoftJson](https://nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson) 库

安装之后可以在 Startup.cs 文件里面的 ConfigureServices 方法添加设置

```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers()
                .AddNewtonsoftJson(options => options.SerializerSettings.ContractResolver = new DefaultContractResolver() { NamingStrategy = new DefaultNamingStrategy() });
        }
```

这样默认的 json 返回值属性使用首字符大写的 PascalCase 属性名风格

使用 DefaultContractResolver 就是 PascalCase 风格

使用 CamelCasePropertyNamesContractResolver 就是 CamelCase 风格

注意，在一些版本，可以是 AddMvc 方法，请看下面

```csharp
services.AddMvc()
    .AddNewtonsoftJson(options =>
           options.SerializerSettings.ContractResolver =
              new DefaultContractResolver());
```

[Migrate from ASP.NET Core 2.2 to 3.0 Preview](https://docs.microsoft.com/en-us/aspnet/core/migration/22-to-30?view=aspnetcore-2.2&tabs=visual-studio )

[Serializing a PascalCase Newtonsoft.Json JObject to camelCase](https://andrewlock.net/serializing-a-pascalcase-newtonsoft-json-jobject-to-camelcase/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
