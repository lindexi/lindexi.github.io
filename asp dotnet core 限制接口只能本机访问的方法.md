# asp dotnet core 限制接口只能本机访问的方法

本文告诉大家，如果限制某些 API 接口，只能让本机进行访问，如只能通过 127.0.0.1 调用某个接口的 get 或 post 方法

<!--more-->

<!-- 发布 -->

官方文档 [Client IP safelist for ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/ip-safelist?view=aspnetcore-5.0) 有告诉大家如何限制 IP 的访问，而本文的需求实际就是限制只能使用本机的 IP 进行访问

先添加一个类 LocalClientIpCheckActionFilter 继承 Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute 类

```csharp
    public class LocalClientIpCheckActionFilter : ActionFilterAttribute
    {
    }
```

在 LocalClientIpCheckActionFilter 重写 OnActionExecuting 方法，这个 OnActionExecuting 方法将会在对应的 API 所在的接口 C# 方法被调用之前被调用。对应的在 ActionFilterAttribute 里面可以重写的，还有 OnActionExecuted 方法，不同之处在于 OnActionExecuting 将会在调用之前被调用，而 OnActionExecuted 将会在被调用之后被调用，细节请参阅 [官方文档](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-5.0#action-filters)

本文是需要在方法被调用之前进行执行的，因此只重写 OnActionExecuting 方法，代码如下

```csharp
        public override void OnActionExecuting(ActionExecutingContext context)
        {
            if (CheckIsLocal(context.HttpContext))
            {
                base.OnActionExecuting(context);
            }
            else
            {
                context.Result = new NotFoundResult();
            }
        }
```

根据官方文档的说明，如果调用了 base.OnActionExecuting 方法，那就是继续执行。而如果给定了 context 的 Result 属性，同时不调用 base.OnActionExecuting 方法，那么将不会继续执行

通过这个特性，咱可以判断，如果是本地的 IP 访问，那么继续执行。否则返回不存在等，请根据你的需求更改 context 的 Result 属性的值

那如何判断当前的访问是本机 IP 访问？可以使用如下代码

```csharp
        private static bool CheckIsLocal(HttpContext httpContext)
        {
            var connection = httpContext.Connection;
            var ipAddress = connection.RemoteIpAddress;
            var localIpAddress = connection.LocalIpAddress;
            if (ipAddress is not null)
            {
                if (localIpAddress != null)
                {
                    return ipAddress.Equals(localIpAddress);
                }
                else
                {
                    return IPAddress.IsLoopback(ipAddress);
                }
            }
            else
            {
                if (localIpAddress == null)
                {
                    // 铁定是 TestServer 或者内转发的
                    return true;
                }
            }

            return false;
        }
```

以上代码需要使用 C# 9.0 或以上版本才能执行

本文的代码其实是我准备做的局域网分享工具 [dotnetCampus.P2PFileShare](https://github.com/dotnet-campus/dotnetCampus.P2PFileShare) 用到的代码，详细请到 [https://github.com/dotnet-campus/dotnetCampus.P2PFileShare](https://github.com/dotnet-campus/dotnetCampus.P2PFileShare) 阅读全部代码

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
