# dotnet 使用 NamedPipeStream 加 MVC 框架的 IPC 库

这是一个用于本机多进程进行 IPC 通讯的库，此库的顶层 API 是采用 ASP.NET Core 的 MVC 框架，其底层通讯不是传统的走网络的方式，而是通过 dotnetCampus.Ipc 开源项目提供的 NamedPipeStream 命名管道的方式进行通讯。本库的优势是可以使用设计非常好的 ASP.NET Core 的 MVC 框架作为顶层调用 API 层，底层通讯采用命名管道可提升传输性能，不走网络可以极大减少网络端口占用问题和减少用户端网络环境带来的问题

<!--more-->
<!-- 发布 -->

<!-- 使用基于 NamedPipeStream 通讯且顶层采用 MVC 框架的 IPC 库 -->

<!-- 的本机多进程通讯 IPC 库 -->

<!-- 命名管道加MVC的本机多进程通讯 IPC 库 -->

## 背景

本机内多进程通讯 IPC 不同于跨设备系统的 RPC 通讯方式，大多数的 IPC 通讯都需要处理复杂的用户端环境问题。对于 RPC 通讯来说，大部分时候，服务端都在开发者完全管控的环境下运行。但 IPC 通讯则无论是服务端还是客户端都可能是在用户端运行的。然而用户端上，无论是系统还是其他环境都是十分复杂的，特别是在国内的，魔改的系统，凶狠的杀毒软件，这些都会让 IPC 通讯受到非预期的打断

传统的 dotnet 系的 IPC 手段有很多个，提供给开发使用的顶层框架也有很多，如 .NET Remoting 和 WCF 等。但是在迁移到 dotnet core 时，由于底层运行时机制的变更，如透明代理不再支持类对象只能支持接口的行为变更，就让 .NET Remoting 从机制性不受支持。为了方便将应用迁移到 dotnet core 框架上，可采用 dotnet campus 组织基于最友好的 MIT 协议开源的 dotnetCampus.Ipc 开源库进行本机内多进程通讯

此 dotnetCampus.Ipc 开源库底层可基于管道进行通讯，经过了约 600 万台设备近半年的测试，发现通过此方式的通讯稳定性极高

无论是 RPC 还是 IPC 通讯，其顶层提供给开发者使用的 API 层，主流上有两个设计阵营。一个是如 .NET Remoting 一样的传输类对象的方式，此方法可以极大隐藏 RPC 或 IPC 的细节，调用远程进程的对象就和调用本机进程一样。另一个阵营是本文的主角，如 ASP.NET Core 的 MVC 模式，通过路由配合参数传递，进行控制器处理的模式，此方式的优良设计已被 ASP.NET Core 所证明，本文也就不多说了

默认下，如此妙的 ASP.NET Core 的 MVC 层框架是仅提供网络传输的方式。然而在诡异的用户端环境下，将有层出不穷的网络通讯问题，如端口被占用，特殊的软件阻止上网等等。让 ASP.NET Core 从走网络的方式，替换为走命名管道的方式，可以极大提升在用户端的稳定性

再次表扬 ASP.NET Core 的优秀设计，在 ASP.NET Core 里，各个模块分层明确，这也就让更换 ASP.NET Core 里的“通讯传输”（其实本意是 IServer 层）这个工作十分简单

在采用 ASP.NET Core 作为 IPC 的顶层调用时，那此时的通讯方式一定就是 服务端-客户端 的形式。服务端可以采用替换 ASP.NET Core 的“通讯传输”为 dotnetCampus.Ipc 的基于命名管道的传输方式。客户端呢？对 ASP.NET Core 来说，最期望客户端的行为是通过 HttpClient 来进行发起调用。刚好 dotnet 下默认的 HttpClient 是支持注入具体的消息传输实现，通过将 dotnetCampus.Ipc 封装为 HttpClient 的消息传输 HttpMessageHandler 就可以让客户端也走 dotnetCampus.Ipc 的传输。如此封装，相当于在 服务端和客户端 的底层传输，全部都在 dotnetCampus.Ipc 层内，分层图如下，通过 dotnetCampus.Ipc 维持稳定的传输从而隐藏具体的 IPC 细节，业务端可以完全复用原有的知识，无须引入额外的 IPC 知识

<!-- 图片 命名管道加MVC的本机多进程通讯 IPC 库.enbx -->

![](http://image.acmx.xyz/lindexi%2F20221161622432592.jpg)

充当 IPC 里的服务端和客户端的业务代码将分别与 ASP.NET Core 和 HttpClient 对接。而 ASP.NET Core 和 HttpClient 又与 dotnetCampus.Ipc 层对接，一切的跨进程通讯逻辑都在 dotnetCampus.Ipc 这一层内完成，由 dotnetCampus.Ipc 层维持稳定的 IPC 传输。下面来看看如何使用此方式开发应用

## 使用方法

接下来将使用 PipeMvcServerDemo 和 PipeMvcClientDemo 这两个例子项目来演示如何使用 ASP.NET Core 的 MVC 层框架加命名管道 NamedPipeStream 做通讯传输的本机内多进程的跨进程通讯 IPC 方式

按照惯例，在 dotnet 系的应用上使用库之前，先通过 NuGet 进行安装。从业务上人为分为服务端和业务端的两个项目，分别安装给服务端用的 [dotnetCampus.Ipc.PipeMvcServer](https://www.nuget.org/packages/dotnetCampus.Ipc.PipeMvcServer) 库，和给客户端用的 [dotnetCampus.Ipc.PipeMvcClient](https://www.nuget.org/packages/dotnetCampus.Ipc.PipeMvcClient) 库

新建的 PipeMvcServerDemo 和 PipeMvcClientDemo 这两个基于 .NET 6 的例子项目都是先基于 WPF 的项目模板创建，从业务上人为分为服务端和业务端的两个项目其实都是运行在相同的一个计算机内，只是为了方便叙述，强行将 PipeMvcServerDemo 称为服务端项目，将 PipeMvcClientDemo 称为客户端项目

### 服务端

先从 PipeMvcServerDemo 服务端项目开始写起，在安装完成 [dotnetCampus.Ipc.PipeMvcServer](https://www.nuget.org/packages/dotnetCampus.Ipc.PipeMvcServer) 库之后，为了使用上 ASP.NET Core 的 MVC 框架，需要在此 WPF 应用里面初始化 ASP.NET Core 框架

初始化的逻辑，和纯放在服务器上的 ASP.NET Core 服务应用只有一点点的差别，那就是在初始化时，需要调用 UsePipeIpcServer 扩展方法，注入 IPC 的服务替换掉默认的 ASP.NET Core 的“通讯传输”（IServer）层。代码如下

```csharp
using dotnetCampus.Ipc.PipeMvcServer;

    private static void RunMvc(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        // 下面一句是关键逻辑
        builder.WebHost.UsePipeIpcServer("PipeMvcServerDemo");

        builder.Services.AddControllers();
        var app = builder.Build();
        app.MapControllers();
        app.Run();
    }
```

调用 UsePipeIpcServer 扩展方法，需要额外加上 `using dotnetCampus.Ipc.PipeMvcServer;` 命名空间。在 UsePipeIpcServer 方法里面需要传入一个参数，此参数用于开启的 IPC 服务所使用的服务名，也就是作为命名管道的管道名。服务名的字符串要求是在当前机器上唯一不重复，推荐采用属性的命名法对其命名传入。此后，客户端的代码即可采用此服务名连接上服务端

也仅仅只需加上 UsePipeIpcServer 扩展方法即可完成对服务端的 IPC 的所有配置

### 客户端

完成服务端的配置之后，可以开始对客户端的配置逻辑，客户端只需要知道服务端的服务名，即如上例子的 `"PipeMvcServerDemo"` 字符串，即可建立和服务端的通讯。在此库的设计上，可以认为服务端的服务名和传统的 C/S 端应用的服务端地址是等同的，至少需要知道服务端的地址才能连接上

在客户端的任意代码里，可采用 IpcPipeMvcClientProvider 提供的 CreateIpcMvcClientAsync 静态方法传入服务名，拿到可以和服务端通讯的 HttpClient 对象，如以下代码

```csharp
using dotnetCampus.Ipc.PipeMvcClient;

        HttpClient ipcPipeMvcClient = await IpcPipeMvcClientProvider.CreateIpcMvcClientAsync("PipeMvcServerDemo");
```

以上代码拿到的 `ipcPipeMvcClient` 对象即可和传统的逻辑一样，进行服务端的请求逻辑，如下文所演示的例子。可以看到客户端的配置逻辑，也只有在初始化时，获取 HttpClient 的逻辑不同

如上面演示的代码，可以看到，无论是客户端还是服务端，初始化的代码都是一句话，没有很多的细节逻辑，方便入手

### 调用

下面开始演示服务端和客户端调用的例子。为了让客户端能调用到客户端对应的服务内容，需要先在服务端创建对应的服务逻辑。以下将演示 GET 和 POST 方法和对应的路由和参数调用方法

在服务端 PipeMvcServerDemo 项目上添加一个 FooController 控制器，代码如下

```csharp
[Route("api/[controller]")]
[ApiController]
public class FooController : ControllerBase
{
    public FooController(ILogger<FooController> logger)
    {
        Logger = logger;
    }

    public ILogger<FooController> Logger { get; }
}
```

在 FooController 添加 Get 方法，代码如下

```csharp
    [HttpGet]
    public IActionResult Get()
    {
        Logger.LogInformation("FooController_Get");
        return Ok(DateTime.Now.ToString());
    }
```

根据 ASP.NET Core 的路由知识，可以在客户端通过 `api/Foo` 路径访问到以上的 Get 方法。接下来编写客户端的逻辑，先在客户端上的 XAML 界面上添加按钮，代码如下

```xml
            <Button x:Name="GetFooButton" Margin="10,10,10,10" Click="GetFooButton_Click">Get</Button>
```

在 `GetFooButton_Click` 方法里面，使用预先拿到的 HttpClient 进行通讯，代码如下

```csharp
using System.Net.Http;

    private async void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        Log($"Start create PipeMvcClient.");

        var ipcPipeMvcClient = await IpcPipeMvcClientProvider.CreateIpcMvcClientAsync("PipeMvcServerDemo");
        _ipcPipeMvcClient = ipcPipeMvcClient;

        Log($"Finish create PipeMvcClient.");
    }

    private HttpClient? _ipcPipeMvcClient;

    private async void GetFooButton_Click(object sender, RoutedEventArgs e)
    {
        if (_ipcPipeMvcClient is null)
        {
            return;
        }

        Log($"[Request][Get] IpcPipeMvcServer://api/Foo");
        var response = await _ipcPipeMvcClient.GetStringAsync("api/Foo");
        Log($"[Response][Get] IpcPipeMvcServer://api/Foo {response}");
    }
```

以上的 Log 方法将输出日志到界面的 TextBlock 控件

以上代码通过 `await _ipcPipeMvcClient.GetStringAsync("api/Foo");` 访问到服务端的 Get 方法，运行效果如下

![](http://image.acmx.xyz/lindexi%2F20221161725566107.jpg)

如上图可以看到，客户端成功调用了服务端，从服务端拿到了返回值

接下来的例子是在 GET 请求带上参数，如实现远程调用计算服务功能，在客户端发送两个 int 数给服务端进行计算相加的值。服务端的代码如下

```csharp
public class FooController : ControllerBase
{
    [HttpGet("Add")]
    public IActionResult Add(int a, int b)
    {
        Logger.LogInformation($"FooController_Add a={a};b={b}");
        return Ok(a + b);
    }
}
```

客户端在 XAML 界面添加对应按钮的代码省略，按钮的事件里调用方法代码如下

```csharp
    private async void GetFooWithArgumentButton_Click(object sender, RoutedEventArgs e)
    {
        Log($"[Request][Get] IpcPipeMvcServer://api/Foo/Add");
        var response = await _ipcPipeMvcClient.GetStringAsync("api/Foo/Add?a=1&b=1");
        Log($"[Response][Get] IpcPipeMvcServer://api/Foo/Add {response}");
    }
```

运行效果如下

![](http://image.acmx.xyz/lindexi%2F20221161746529208.jpg)

可以看到客户端成功调用了服务端执行了计算，拿到了返回值

通过以上的例子可以看到，即使底层更换为 IPC 通讯，对于上层业务代码，调用服务端的逻辑，依然没有引入任何新的 IPC 知识，都是对 HttpClient 的调用

接下来是 POST 调用的代码，服务端在 FooController 类上添加 Post 方法，加上 HttpPostAttribute 特性，代码如下

```csharp
    [HttpPost]
    public IActionResult Post()
    {
        Logger.LogInformation("FooController_Post");
        return Ok($"POST {DateTime.Now}");
    }
```

客户端编写 PostFooButton 按钮，在按钮点击事件添加如下代码用于请求服务端

```csharp
    private async void PostFooButton_Click(object sender, RoutedEventArgs e)
    {
        Log($"[Request][Post] IpcPipeMvcServer://api/Foo");
        var response = await _ipcPipeMvcClient.PostAsync("api/Foo", new StringContent(""));
        var m = await response.Content.ReadAsStringAsync();
        Log($"[Response][Post] IpcPipeMvcServer://api/Foo {response.StatusCode} {m}");
    }
```

运行效果如下图

![](http://image.acmx.xyz/lindexi%2F20221161750511785.jpg)

如上图可以看到客户端成功采用 POST 方法请求到服务端

接下来将采用 POST 方法带参数方式请求服务端，服务端处理客户端请求过来的参数执行实际的业务逻辑，服务端的代码依然放在 FooController 类里

```csharp
    [HttpPost("PostFoo")]
    public IActionResult PostFooContent(FooContent foo)
    {
        Logger.LogInformation($"FooController_PostFooContent Foo1={foo.Foo1};Foo2={foo.Foo2 ?? "<NULL>"}");
        return Ok($"PostFooContent Foo1={foo.Foo1};Foo2={foo.Foo2 ?? "<NULL>"}");
    }
```

以上代码采用 FooContent 作为参数，类型定义如下

```csharp
public class FooContent
{
    public string? Foo1 { set; get; }
    public string? Foo2 { set; get; }
}
```

客户端代码如下，为了给出更多细节，我将不使用 PostAsJsonAsync 方法，而是先创建 FooContent 对象，将 FooContent 对象序列化为 json 字符串，再 POST 请求

```csharp
    private async void PostFooWithArgumentButton_Click(object sender, RoutedEventArgs e)
    {
        Log($"[Request][Post] IpcPipeMvcServer://api/Foo");

        var json = JsonSerializer.Serialize(new FooContent
        {
            Foo1 = "Foo PostFooWithArgumentButton",
            Foo2 = null,
        });
        StringContent content = new StringContent(json, Encoding.UTF8, "application/json");
        var response = await _ipcPipeMvcClient.PostAsync("api/Foo/PostFoo", content);
        var m = await response.Content.ReadAsStringAsync();
        Log($"[Response][Post] IpcPipeMvcServer://api/Foo/PostFoo {response.StatusCode} {m}");
    }
```

运行效果如下图

![](http://image.acmx.xyz/lindexi%2F20221161816537986.jpg)

如上图，客户端成功将 FooContent 参数传给服务端

以上就是 GET 和 POST 的例子，几乎看不出来加上 IPC 前后对 ASP.NET Core 应用调用的差别，除了要求需要使用特定的 HttpClient 对象之外，其他的逻辑都相同。以上的例子项目，可以从本文末尾获取

如关注此库的实现原理，请继续阅读下文

## 原理

先从客户端方向开始，在客户端里使用的 HttpClient 是被注入了使用 IPC 底层框架通讯的 IpcNamedPipeClientHandler 对象，此 IpcNamedPipeClientHandler 对象是一个继承 HttpMessageHandler 类型的对象

在 IpcNamedPipeClientHandler 重写了 HttpMessageHandler 类型的 SendAsync 方法，可以让所有使用 HttpClient 发送的请求，进入 IpcNamedPipeClientHandler 的逻辑。在此方法里面，将序列化请求，将请求通过 dotnetCampus.Ipc 发送到服务端，再通过 dotnetCampus.Ipc 提供的消息请求机制，等待收到服务端对此请求的返回值。等收到服务端的返回值之后，封装成为 HttpResponseMessage 返回值，让此返回值接入到 HttpClient 的机制框架，从而实现调用 HttpClient 发送的请求是通过 dotnetCampus.Ipc 层传输而不是走网络。进入 dotnetCampus.Ipc 层是被设计为对等层，对客户端来说，进入 dotnetCampus.Ipc 层具体是走到 ASP.NET Core 的 MVC 或者是其他框架都是不需要关注的。对客户端来说，只需要知道进入 dotnetCampus.Ipc 层的请求，可以进行异步等待请求，细节逻辑不需要关注

以下是 IpcNamedPipeClientHandler 的实现代码

```csharp
    class IpcNamedPipeClientHandler : HttpMessageHandler
    {
        protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
        	// 序列化请求消息，准备通过 IPC 层发送
            var message = HttpMessageSerializer.Serialize(request);

            // 创建 IPC 消息的 Tag 内容，此 Tag 内容仅用来调试和记录日志
            var ipcMessageTag = request.RequestUri?.ToString() ?? request.Method.ToString();

            // 在 dotnetCampus.Ipc 层，采用 P2P 模型，没有具体的服务端和客户端
            // 但是 P2P 模型是可以模拟 C/S 模型的，只需要让某个端（Peer）充当服务端，另外的端充当客户端即可
            // 在 dotnetCampus.Ipc 库里，采用 PeerProxy 表示某个端
            // 这里的端表示的是 IPC 的某个端，大部分时候可以认为是一个进程
            // 以下的 ServerProxy 就是充当服务端的一个端，将在此框架内被初始化创建

            // 通过 PeerProxy 发送 IPC 请求，此时的 IPC 请求将会被 PipeMvcServer 处理
            // 在 PipeMvcServer 里面，将通过 ASP.NET Core MVC 框架层进行调度，分发到对应的控制器处理
            // 控制器处理完成之后，将由 MVC 框架层将控制器的输出交给 PipeMvcServer 层
            // 在 PipeMvcServer 层收到控制器的输出之后，将通过 IPC 框架，将输出返回给 PipeMvcClient 端
            // 当 PipeMvcClient 收到输出返回值后，以下的 await 方法将会返回
            var response = await ServerProxy.GetResponseAsync(new IpcMessage(ipcMessageTag, message));

            // 将 IPC 返回的消息反序列化为 HttpResponseMessage 用于接入 HttpClient 框架
            return HttpMessageSerializer.DeserializeToResponse(response.Body);
        }

        private PeerProxy ServerProxy { get; }

        // 忽略其他代码
    }
```

这就是为什么客户端需要通过 IpcPipeMvcClientProvider 的 CreateIpcMvcClientAsync 拿到 HttpClient 的原因。在 CreateIpcMvcClientAsync 方法，不仅需要创建 HttpClient 对象，还需要先尝试连接服务端。尽管从 HttpClient 的设计上，应该是发起请求时才去连接服务端，但因为这是 IPC 通讯，且为了解决 IPC 初始化逻辑的多进程资源竞争，当前版本采用在获取 HttpClient 也就是发起具体请求之间，连接服务端

```csharp
    /// <summary>
    /// 提供给客户端调用 MVC 的 Ipc 服务的功能
    /// </summary>
    public static class IpcPipeMvcClientProvider
    {
        /// <summary>
        /// 获取访问 Mvc 的 Ipc 服务的对象
        /// </summary>
        /// <param name="ipcPipeMvcServerName">对方 Ipc 服务名</param>
        /// <param name="clientIpcProvider">可选，用来进行 Ipc 连接的本地服务。如不传或是空，将创建新的 Ipc 连接服务</param>
        /// <returns></returns>
        public static async Task<HttpClient> CreateIpcMvcClientAsync(string ipcPipeMvcServerName, IpcProvider? clientIpcProvider = null)
        {
            if (clientIpcProvider == null)
            {
                clientIpcProvider = new IpcProvider();
                clientIpcProvider.StartServer();
            }

            var peer = await clientIpcProvider.GetAndConnectToPeerAsync(ipcPipeMvcServerName);

            return new HttpClient(new IpcNamedPipeClientHandler(peer, clientIpcProvider))
            {
                BaseAddress = new Uri(IpcPipeMvcContext.BaseAddressUrl),
            };
        }
    }
```

在 dotnetCampus.Ipc 层是采用 P2P 方式设计的，因此客户端也需要创建自己的 IpcProvider 对象。客户端可选传入已有的 IpcProvider 对象进行复用，就如 HttpClient 复用逻辑一样。但创建 IpcProvider 对象是很便宜的，不会占用多少资源，是否复用在性能上没有多少影响。但是支持传入 IpcProvider 更多是可以方便开发者对 IpcProvider 进行的定制逻辑，例如注入自己的数组池和日志等

以上就是客户端的逻辑。关于如何序列化请求消息等，这些就属于细节了，无论采用什么方法，只需要能将请求和响应与二进制 byte 数组进行序列化和反序列化即可。细节内容还请自行在本文末尾获取源代码进行阅读

服务端的逻辑相对复杂一些，在服务端的 dotnetCampus.Ipc 层收到客户端的请求后，服务端将构建一个虚拟的访问请求，此访问请求将通过 继承 IServer 接口的 IpcServer 对象，在 ASP.NET Core 框架内发起请求，通过 MVC 框架层处理之后将响应返回到 IpcServer 对象里交给 dotnetCampus.Ipc 层传输给客户端

在 IpcServer 对象的启动函数，也就是 StartAsync 函数里面，将会同步初始化 IpcPipeMvcServerCore 对象。在 IpcPipeMvcServerCore 对象里面将初始化创建 dotnetCampus.Ipc 层的通讯机制。代码如下

```csharp
    public class IpcServer : IServer
    {
        public IpcServer(IServiceProvider services, IFeatureCollection featureCollection, IOptions<IpcServerOptions> optionsAccessor)
        {
        	// 忽略代码
            var ipcCore = Services.GetRequiredService<IpcPipeMvcServerCore>();
            IpcPipeMvcServerCore = ipcCore;
        }

        Task IServer.StartAsync<TContext>(IHttpApplication<TContext> application, CancellationToken cancellationToken)
        {
            // 忽略代码
            IpcPipeMvcServerCore.Start();
        }

        private IpcPipeMvcServerCore IpcPipeMvcServerCore { get; }

        // 忽略代码
    }
```

而 IpcPipeMvcServerCore 和 IpcServer 对象都是在调用 `builder.WebHost.UsePipeIpcServer(xxx);` 被注入，如以下代码

```csharp
    public static class WebHostBuilderExtensions
    {
        /// <summary>
        /// Enables the <see cref="IpcServer" /> service. 启用命名管道IPC服务
        /// </summary>
        /// <param name="builder">The <see cref="IWebHostBuilder"/>.</param>
        /// <param name="ipcPipeName">设置 Ipc 服务的管道名</param>
        /// <returns>The <see cref="IWebHostBuilder"/>.</returns>
        public static IWebHostBuilder UsePipeIpcServer(this IWebHostBuilder builder, string ipcPipeName)
        {
            return builder.ConfigureServices(services =>
            {
            	// 忽略代码
                services.AddSingleton<IServer, IpcServer>();

                services.AddSingleton<IpcPipeMvcServerCore>(s => new IpcPipeMvcServerCore(s, ipcPipeName));
            });
        }
    }
```

依靠 ASP.NET Core 的机制，将会在主机启动，调用 IServer 的 StartAsync 方法。通过 IpcServer 的 StartAsync 方法启动 IpcPipeMvcServerCore 的逻辑

在 IpcPipeMvcServerCore 里，将初始化 IpcProvider 服务。这里的 IpcProvider 服务是 dotnetCampus.Ipc 提供的服务对外的接口，通过 IpcProvider 可以和 dotnetCampus.Ipc 层的其他 Peer 进行通讯。刚好在客户端也相同的初始化 IpcProvider 服务，通过 `ipcPipeName` 管道名可以将客户端和服务端关联

```csharp
    class IpcPipeMvcServerCore
    {
        public IpcPipeMvcServerCore(IServiceProvider serviceProvider, string? ipcServerName)
        {
            ipcServerName ??= "IpcPipeMvcServer" + Guid.NewGuid().ToString("N");

            IpcServer = new IpcProvider(ipcServerName, new IpcConfiguration()
            {
                DefaultIpcRequestHandler = new DelegateIpcRequestHandler(async context =>
                {
                    // 核心代码
                })
            });
        }

        public void Start() => IpcServer.StartServer();
        public IpcProvider IpcServer { set; get; }
    }
```

在 dotnetCampus.Ipc 层提供了请求响应框架，可以通过传入 DefaultIpcRequestHandler 对象用来接收其他端发送过来的请求，处理完成之后返回给对方。上面代码的核心就是 DelegateIpcRequestHandler 的处理逻辑，在 context 里读取客户端的请求信息，反序列化为 HttpRequestMessage 对象，通过内部逻辑进入到 ASP.NET Core 层，再通过 MVC 框架之后拿到请求的返回值，将返回值封装为 IpcResponseMessageResult 返回给客户端

```csharp
            IpcServer = new IpcProvider(ipcServerName, new IpcConfiguration()
            {
                DefaultIpcRequestHandler = new DelegateIpcRequestHandler(async context =>
                {
                	// 将请求反序列化为 HttpRequestMessage 对象
                	// 用于传入到 ASP.NET Core 层
                    System.Net.Http.HttpRequestMessage? requestMessage = HttpMessageSerializer.DeserializeToRequest(context.IpcBufferMessage.Body);

                    // 创建虚拟的请求，进入到 ASP.NET Core 框架里
                    var server = (IpcServer) serviceProvider.GetRequiredService<IServer>();
                    var clientHandler = (ClientHandler) server.CreateHandler();
                    var response = await clientHandler.SendInnerAsync(requestMessage, CancellationToken.None);

                    // 拿到的返回值序列化为 IpcResponseMessageResult 放入 dotnetCampus.Ipc 层用来返回客户端
                    var responseByteList = HttpMessageSerializer.Serialize(response);
                    return new IpcResponseMessageResult(new IpcMessage($"[Response][{requestMessage.Method}] {requestMessage.RequestUri}", responseByteList));
                })
            });
```

创建虚拟的请求，进入 ASP.NET Core 框架里的逻辑是服务端最复杂的部分。在 IpcServer 的 CreateHandler 方法里面，将创建 ClientHandler 对象。此 ClientHandler 对象是用来构建虚拟的请求，相当于在当前进程内发起请求而不是通过网络层发起请求，代码如下

```csharp
    public class IpcServer : IServer
    {
        /// <summary>
        /// Creates a custom <see cref="HttpMessageHandler" /> for processing HTTP requests/responses with the test server.
        /// </summary>
        public HttpMessageHandler CreateHandler()
        {
        	// 忽略代码
            return new ClientHandler(BaseAddress, Application) { AllowSynchronousIO = AllowSynchronousIO, PreserveExecutionContext = PreserveExecutionContext };
        }
    }
```

在也是继承 HttpMessageHandler 的 ClientHandler 里，也重写了 SendInnerAsync 方法，此方法将会负责创建 HttpContextBuilder 对象，由 HttpContextBuilder 执行具体的调用 ASP.NET Core 层的逻辑

```csharp
        public async Task<HttpResponseMessage> SendInnerAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
       
            // 创建 HttpContextBuilder 对象
            var contextBuilder = new HttpContextBuilder(_application, AllowSynchronousIO, PreserveExecutionContext);

            var requestContent = request.Content;

            if (requestContent != null)
            {
            	// 以下是对 HttpContextBuilder 的初始化逻辑

                // Read content from the request HttpContent into a pipe in a background task. This will allow the request
                // delegate to start before the request HttpContent is complete. A background task allows duplex streaming scenarios.
                contextBuilder.SendRequestStream(async writer =>
                {
                    // 忽略初始化逻辑
                });
            }

            contextBuilder.Configure((context, reader) =>
            {
                // 忽略初始化逻辑
            });

            // 忽略其他代码

            // 执行实际的调用 ASP.NET Core 框架逻辑
            var httpContext = await contextBuilder.SendAsync(cancellationToken);

            // 创建 HttpResponseMessage 对象用于返回
            var response = new HttpResponseMessage();

            // 以下是对 HttpResponseMessage 的初始化逻辑，从 httpContext 里获取返回值
            response.StatusCode = (HttpStatusCode) httpContext.Response.StatusCode;
            response.ReasonPhrase = httpContext.Features.Get<IHttpResponseFeature>()!.ReasonPhrase;
            response.RequestMessage = request;
            response.Version = request.Version;
            response.Content = new StreamContent(httpContext.Response.Body);

            // 忽略其他代码

            return response;
        }
```

在 HttpContextBuilder 里，将在 SendAsync 逻辑里调用 ApplicationWrapper 的 ProcessRequestAsync 方法从而调入 ASP.NET Core 框架内。这里的 ApplicationWrapper 是对 `Microsoft.AspNetCore.Hosting.HostingApplication` 的封装，因为此 HostingApplication 类型是不对外公开的。以上这几个类型的定义逻辑，都是现有的 [https://github.com/dotnet/aspnetcore](https://github.com/dotnet/aspnetcore) 开源仓库的代码

通过当前进程发起请求而不通过网络层的逻辑，其实在 ASP.NET Core 开源仓库里面有默认的一个实现的提供。那就是为了单元测试编写的 TestHost 机制

在 TestHost 机制里，开发者可以在单元测试里面开启 ASP.NET Core 主机，但是不需要监听任何网络的端口，所有对此主机的测试完全通过 TestHost 机制走进程内的模拟请求发起。对于业务代码来说，大多数时候不需要关注请求的发起方具体是谁，因此单元测试上可以使用 TestHost 方便进行测试业务代码，或者是在集成测试上测试调用逻辑。使用 TestHost 可以让单元测试或集成测试不需要关注网络的监听，防止测试错服务，方便在 CI 里加入测试逻辑

刚好此机制的代码也是本库所需要的，通过拷贝了 [https://github.com/dotnet/aspnetcore](https://github.com/dotnet/aspnetcore) 开源仓库的关于 TestHost 的机制代码，即可用来实现 IpcServer 的逻辑

也如放在 IpcServer 的 CreateHandler 函数上的代码注释，这就是原本的 TestHost 里对应函数的代码

相当于在 TestHost 机制上再加上一层，这一层就是基于 dotnetCampus.Ipc 层做通讯，通过 TestHost 层创建虚拟的请求，进入 ASP.NET Core 框架

为了方便开发者接入，也为了防止开发者接入了 dotnetCampus.Ipc 层的 IpcNamedPipeStreamMvcServer 之后，再接入 TestHost 进行单元测试的冲突，本仓库更改了所有从 [https://github.com/dotnet/aspnetcore](https://github.com/dotnet/aspnetcore) 开源仓库的关于 TestHost 的机制代码的命名空间，对入口调用函数和类型也进行重命名。在每个拷贝的文件上都加上了 `// Copy From: https://github.com/dotnet/aspnetcore` 的注释

## 代码

本文所有代码都放在 [https://github.com/dotnet-campus/dotnetCampus.Ipc](https://github.com/dotnet-campus/dotnetCampus.Ipc) 开源仓库里，欢迎访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
