
# ASP.NET Core 6 浅析环境变量与配置的优先级

本文将简单介绍在 dotnet 6 下的 ASP.NET Core 里的环境变量当成配置的优先级行为。这部分内容在官方文档都有提到，只是我开始粗心没有看仔细，而踩到了坑，特意写篇博客记录一下

<!--more-->


<!-- CreateTime:2023/6/25 8:43:13 -->
<!-- 标题： ASP.NET Core 浅析环境变量与配置的优先级 -->
<!-- 发布 -->
<!-- 博客 -->

在 ASP.NET Core 环境变量和 appsettings.json 文件的优先级从高到低如下：

- 不带 `ASPNETCORE_` 前缀的环境变量
- appsettings.json
- 带 `ASPNETCORE_` 前缀的环境变量

本文将使用日志配置作为例子来告诉大家 ASP.NET Core 的行为

在默认的项目配置里面，存放在 appsettings.json 文件里面有关于日志的默认配置，代码如下

```json
{
  "Logging": 
  {
    "LogLevel": 
    {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}

```

通过以上的 `"Microsoft.AspNetCore": "Warning"` 日志配置可以知道，如果创建了使用 `Microsoft.AspNetCore` 前缀的 CategoryName 的 ILogger 日志，那么这个日志只有大于等于 Warning 等级的日志才能输出。而创建了其他前缀的 CategoryName 的 ILogger 日志，将可以采用 `"Default": "Information"` 配置，让大于等于 Information 等级的日志进行输出

如在 Program.cs 编写以下代码，尝试进行日志输出

```csharp
// 配置输出到 VisualStudio 调试界面
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddLogging(loggingBuilder => loggingBuilder.AddDebug());

var app = builder.Build();

app.MapGet("/", () => "Hello World!");

var serviceProvider = app.Services;

var loggerFactory = serviceProvider.GetRequiredService<ILoggerFactory>();
var logger1 = loggerFactory.CreateLogger(categoryName: "Microsoft.AspNetCore.Foo");
logger1.LogInformation($"Logger1 LogInfo");
logger1.LogWarning($"Logger1 LogWarning");

var logger2 = loggerFactory.CreateLogger("Microsoft.Foo");
logger2.LogInformation($"Logger2 LogInfo");
logger2.LogWarning($"Logger2 LogWarning");
```

此时符合预期的就是 `logger1` 只输出 Warning 信息，而 `logger2` 输出两条日志。运行程序，在 VisualStudio 的调试输出窗口可以看到以下输出

```csharp
Microsoft.AspNetCore.Foo: Warning: Logger1 LogWarning
Microsoft.Foo: Information: Logger2 LogInfo
Microsoft.Foo: Warning: Logger2 LogWarning
```

接下来尝试设置环境变量修改其优先级，根据官方文档可以知道，环境变量设置里面使用 `__` 代替 `:` 符号，即在设置 `Logging:LogLevel:Microsoft.AspNetCore` 时可以采用 `Logging__LogLevel__Microsoft.AspNetCore` 这样的 Key 进行设置

设置环境变量的另一个方式是通过在环境变量前面加上 `ASPNETCORE_` 前缀，换句话可以使用 `ASPNETCORE_Logging__LogLevel__Microsoft.AspNetCore` 这样的配置

通过本文开始可以知道的是环境变量这两个不同的格式和 appsettings.json 文件的优先级不同。带 `ASPNETCORE_` 前缀的环境变量优先级低于不带的 `ASPNETCORE_` 前缀和配置文件的

为了测试环境变量，自然不能让大家去改本机的环境变量了，只需要在 VisualStudio 调试配置里面编辑即可，可以直接编辑项目的 `Properties\launchSettings.json` 文件，如本文的测试例子，修改为如下代码

```json
{
  "profiles": 
  {
    "IIS Express": 
    {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": 
      {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "配置文件 1": 
    {
      "commandName": "Project",
      "environmentVariables": 
      {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "ASPNETCORE_Logging__LogLevel__Microsoft.AspNetCore": "Debug",
        "Logging__LogLevel__Microsoft": "Warning"
      },
      "dotnetRunMessages": true,
      "applicationUrl": "http://localhost:5124"
    },
    "配置文件 2": 
    {
      "commandName": "Project",
      "environmentVariables": 
      {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "Logging__LogLevel__Microsoft.AspNetCore": "Debug",
        "ASPNETCORE_Logging__LogLevel__Microsoft": "Warning"
      },
      "dotnetRunMessages": true,
      "applicationUrl": "http://localhost:5124"
    }
  },
  "iisSettings": 
  {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": 
    {
      "applicationUrl": "http://localhost:54586",
      "sslPort": 0
    }
  }
}
```

修改 Program.cs 代码如下

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddLogging(loggingBuilder => loggingBuilder.AddDebug());

var app = builder.Build();

app.MapGet("/", () => "Hello World!");

var logLevelConfiguration = app.Configuration.GetSection("Logging").GetSection("LogLevel");
var configuration1 = logLevelConfiguration["Microsoft.AspNetCore"];
var configuration2 = logLevelConfiguration["Microsoft"];

// 配置文件 1：
// configuration1 = Warning
// configuration2 = Warning

// 配置文件 2：
// configuration1 = Debug
// configuration2 = Warning

var serviceProvider = app.Services;

var loggerFactory = serviceProvider.GetRequiredService<ILoggerFactory>();
var logger1 = loggerFactory.CreateLogger(categoryName: "Microsoft.AspNetCore.Foo");
logger1.LogInformation($"Logger1 LogInfo");
logger1.LogWarning($"Logger1 LogWarning");

var logger2 = loggerFactory.CreateLogger("Microsoft.Foo");
logger2.LogInformation($"Logger2 LogInfo");
logger2.LogWarning($"Logger2 LogWarning");

//app.Run();

Console.Read();
```

尝试切换配置文件 1 测试其行为，可以看到在配置文件 1 的 VisualStudio 控制台输出如下

```
Microsoft.AspNetCore.Foo: Warning: Logger1 LogWarning
Microsoft.Foo: Warning: Logger2 LogWarning
```

这是因为在配置1里面的配置分别是 `"ASPNETCORE_Logging__LogLevel__Microsoft.AspNetCore": "Debug"` 和 `"Logging__LogLevel__Microsoft": "Warning"` 导致了 `Microsoft.AspNetCore` 的配置优先级低于配置文件的配置，从而没有成功赋值上 Debug 等级

切换到配置文件 2 测试其行为，可以看到在配置文件 2 的 VisualStudio 控制台输出如下

```
Microsoft.AspNetCore.Foo: Information: Logger1 LogInfo
Microsoft.AspNetCore.Foo: Warning: Logger1 LogWarning
Microsoft.Foo: Warning: Logger2 LogWarning
```

可以看到 Logger1 输出了 Info 和 Warning 等级的日志，这就表示了通过 `Logging__LogLevel__Microsoft.AspNetCore` 环境变量设置的配置的优先级高于配置文件

以上测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/91debf6e102bb11f8ecfd290c0951827744254d2/JayabawwiWhenenearfajay) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/91debf6e102bb11f8ecfd290c0951827744254d2/JayabawwiWhenenearfajay) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个名为 JayabawwiWhenenearfajay 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 91debf6e102bb11f8ecfd290c0951827744254d2
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 91debf6e102bb11f8ecfd290c0951827744254d2
```

获取代码之后，进入 JayabawwiWhenenearfajay 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。