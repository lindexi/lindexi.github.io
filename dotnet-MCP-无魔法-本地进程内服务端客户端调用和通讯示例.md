
# dotnet MCP 无魔法 本地进程内服务端客户端调用和通讯示例

官方给的 MCP 示例都是带上 AI 魔法的，或者要么就是控制台或进程调用等，不利于我了解 MCP 的机制。本文记录采用本地进程内的 MCP 服务端和客户端相互通讯的方式，方便大家了解 MCP 的基础机制

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

本文核心重点在 ITransport 接口上。在 <https://github.com/modelcontextprotocol/csharp-sdk> 给出的官方示例里面，服务端的通讯现在（2025-05-09）还是使用控制台方式作为输入，客户端的部分采用的是启动 Everything 进程。这个过程不利于我在一个进程内，让 MCP 客户端调用同进程内的 MCP 服务端，让我了解其工作机制

创建 MCP 服务端用到的 `McpServerFactory.Create` 方法需要传入两个参数，其方法签名定义如下

```csharp
public static class McpServerFactory
{
    public static IMcpServer Create
    (
        ITransport transport,
        McpServerOptions serverOptions,
        ILoggerFactory? loggerFactory = null,
        IServiceProvider? serviceProvider = null
    )
    ...
}
```

第一个参数就是上文提到的 ITransport 类型参数，用于决定数据的传入和输出。其接口定义如下

```csharp
public interface ITransport : IAsyncDisposable
{
    ChannelReader<JsonRpcMessage> MessageReader { get; }

    Task SendMessageAsync(JsonRpcMessage message, CancellationToken cancellationToken = default);
}
```

这个接口的定义看起来非常舒服，其意图就是从 MessageReader 让 MCP 服务端收到输入。当 MCP 服务端能够完成响应时，通过 SendMessageAsync 将数据进行输出

再来看看 MCP 客户端的创建 `McpClientFactory.CreateAsync` 方法，这个方法里面包含了将 MCP 客户端创建出来且和 MCP 服务端完成连接的过程

```csharp
public static partial class McpClientFactory
{
    public static async Task<IMcpClient> CreateAsync
    (
        IClientTransport clientTransport,
        McpClientOptions? clientOptions = null,
        ILoggerFactory? loggerFactory = null,
        CancellationToken cancellationToken = default
    )
    ...
}
```

再进一步看看 `McpClientFactory.CreateAsync` 方法的第一个参数 `IClientTransport` 接口类型的定义

```csharp
public interface IClientTransport
{
    string Name { get; }

    Task<ITransport> ConnectAsync(CancellationToken cancellationToken = default);
}
```

可以看到 IClientTransport 和 ITransport 的不同在于 IClientTransport 是等待连接之后再返回 ITransport 对象

了解到这里，大概也就能明白了如何在同一个进程内的示例代码编写思路了

在同进程内，客户端连接服务端的 `IClientTransport.ConnectAsync` 肯定是瞬间完成的，即可以将其约掉，等同于客户端 `McpClientFactory.CreateAsync` 方法也要有一个 ITransport 接口对象传入

如此可以看到 MCP 客户端和服务端的 ITransport 正好可以传入一对对象，让一方的输入成为另一方的输出，让另一方的输出成为这一方的输入。如此就可以完成通讯逻辑

为了简单起见，我编写了名为 InterprocessTransportFactory 的辅助类，其作用就是创建两个 `Channel<JsonRpcMessage>` 对象，分别给到客户端和服务端，代码如下

```csharp
class InterprocessTransportFactory
{
    public InterprocessTransportFactory()
    {
        _clientTransport = new ClientTransport(this);
        _serverTransport = new ServerTransport(this);
    }

    private readonly ClientTransport _clientTransport;
    private readonly ServerTransport _serverTransport;

    public IClientTransport GetClientTransport()
        => _clientTransport;

    public ITransport GetServerTransport()
        => _serverTransport;

    class ClientTransport(InterprocessTransportFactory factory) : TransportBase, IClientTransport, ITransport
    {
        private readonly InterprocessTransportFactory _factory = factory;

        public Task<ITransport> ConnectAsync(CancellationToken cancellationToken = new CancellationToken())
        {
            return Task.FromResult<ITransport>(this);
        }

        public string Name => "ClientTransport";

        public override async Task SendMessageAsync(JsonRpcMessage message,
            CancellationToken cancellationToken = new CancellationToken())
        {
            await _factory._serverTransport.Channel.Writer.WriteAsync(message, cancellationToken);
        }
    }

    class ServerTransport(InterprocessTransportFactory factory) : TransportBase
    {
        private readonly InterprocessTransportFactory _factory = factory;

        public override async Task SendMessageAsync(JsonRpcMessage message,
            CancellationToken cancellationToken = new CancellationToken())
        {
            await _factory._clientTransport.Channel.Writer.WriteAsync(message, cancellationToken);
        }
    }

    abstract class TransportBase : ITransport
    {
        public Channel<JsonRpcMessage> Channel { get; } =
            System.Threading.Channels.Channel.CreateUnbounded<JsonRpcMessage>();

        public ValueTask DisposeAsync()
        {
            Channel.Writer.Complete();
            return ValueTask.CompletedTask;
        }

        public abstract Task SendMessageAsync(JsonRpcMessage message,
            CancellationToken cancellationToken = new CancellationToken());

        public ChannelReader<JsonRpcMessage> MessageReader => Channel.Reader;
    }
}
```

核心逻辑就是在 ClientTransport 的 SendMessageAsync 方法，通过 `await _factory._serverTransport.Channel.Writer.WriteAsync(message, cancellationToken);` 写入到服务端，作为服务端的输入。反之，在服务端的 ServerTransport.SendMessageAsync 方法，也将输出写入到客户端

如以上代码所示，由于是在相同的进程内，客户端连接是瞬时发生的，对应到代码里面就是 `ClientTransport.ConnectAsync` 方法直接返回自身

完成基础逻辑之后，以下思路就是先创建服务端，再创建客户端，让客户端连接上服务端，且尝试调用服务端的工具

创建服务端需要有两个必要参数，分别是 ITransport 和 McpServerOptions 类型的参数。其中 ITransport 已经在上文准备好了，接下来继续看一下 McpServerOptions 类型的参数

按照设计，将在 McpServerOptions 参数里面放入工具的枚举，即列举出当前服务端有哪些工具的处理，以及收到客户端调用工具时的处理逻辑。这两个处理逻辑底层只是两个委托而已，具体如何实现都需要看咱自己处理，其代码如下

```csharp
McpServerOptions options = new()
{
    ServerInfo = new Implementation() { Name = "MyServer", Version = "1.0.0" },
    Capabilities = new ServerCapabilities()
    {
        Tools = new ToolsCapability()
        {
            ListToolsHandler = (request, cancellationToken) =>
                ...,// 返回工具列表

            CallToolHandler = (request, cancellationToken) =>
                ...,// 处理调用工具
        }
    },
};
```

框架上层的封装，如官方示例的以下代码，也仅仅只是帮忙封装 ListToolsHandler 和 CallToolHandler 两个属性而已

```csharp
[McpServerToolType]
public static class EchoTool
{
    [McpServerTool, Description("Echoes the message back to the client.")]
    public static string Echo(string message) => $"hello {message}";
}
```

按照官方给的无魔法的实现方式，其示例代码如下

```csharp
McpServerOptions options = new()
{
    ServerInfo = new Implementation() { Name = "MyServer", Version = "1.0.0" },
    Capabilities = new ServerCapabilities()
    {
        Tools = new ToolsCapability()
        {
            ListToolsHandler = (request, cancellationToken) =>
                ValueTask.FromResult(new ListToolsResult()
                {
                    Tools =
                    [
                        new Tool()
                        {
                            Name = "echo",
                            Description = "Echoes the input back to the client.",
                            InputSchema = JsonSerializer.Deserialize<JsonElement>("""
                                {
                                    "type": "object",
                                    "properties": {
                                      "message": {
                                        "type": "string",
                                        "description": "The input to echo back"
                                      }
                                    },
                                    "required": ["message"]
                                }
                                """),
                        }
                    ]
                }),

            CallToolHandler = (request, cancellationToken) =>
            {
                if (request.Params?.Name == "echo")
                {
                    if (request.Params.Arguments?.TryGetValue("message", out var message) is not true)
                    {
                        throw new McpException("Missing required argument 'message'");
                    }

                    return ValueTask.FromResult(new CallToolResponse()
                    {
                        Content = [new Content() { Text = $"Echo: {message}", Type = "text" }]
                    });
                }

                throw new McpException($"Unknown tool: '{request.Params?.Name}'");
            },
        }
    },
}
```

可以看到在没有魔法的帮助下，只是简单写一个 echo 方法就需要很好代码量

准备好了 McpServerOptions 和 ITransport 类型的对象之后，就可以创建和启动 MCP 服务端了，代码如下

```csharp
var transportFactory = new InterprocessTransportFactory();

await using IMcpServer server = McpServerFactory.Create(transportFactory.GetServerTransport(), options);
_ = server.RunAsync();
```

相对比来说，客户端的代码就简单许多了，只需简单从 InterprocessTransportFactory 拿到 IClientTransport 即可调用 `McpClientFactory.CreateAsync` 方法创建 MCP 客户端，如以下代码所示

```csharp
var client = await McpClientFactory.CreateAsync(transportFactory.GetClientTransport());
```

以上代码即可完成创建 MCP 客户端，且让 MCP 客户端连接上 MCP 服务端

连接完成之后，可以使用 `IMcpClient.ListToolsAsync` 列举出所连接的 MCP 服务端提供的工具列表，代码如下

```csharp
// Print the list of tools available from the server.
foreach (var tool in await client.ListToolsAsync())
{
    Console.WriteLine($"{tool.Name} ({tool.Description})");
}
```

在本示例里面，会在控制台输出以下内容

```
echo (Echoes the input back to the client.)
```

既然枚举到了工具，那接下来就尝试调用一下工具，代码如下

```csharp
var dictionary = new Dictionary<string, object?>()
{
    { "message", "Hello, World!" }
};
CallToolResponse callToolResponse = await client.CallToolAsync("echo", dictionary);
foreach (var content in callToolResponse.Content)
{
    Console.WriteLine($"CallToolResponse: Type={content.Type} Text='{content.Text}'");
}
```

继续运行以上代码，将在控制台输出以下内容

```
CallToolResponse: Type=text Text='Echo: Hello, World!'
```

在执行 `await client.CallToolAsync("echo", dictionary)` 代码之前，在 ToolsCapability 的 CallToolHandler 委托打上断点，即可通过调用堆栈看到整个调用过程，如以下调用堆栈所示

```
 	ModelContextProtocol.dll!ModelContextProtocol.Server.McpServer.InvokeHandlerAsync.__InvokeScopedAsync|0(System.Func<ModelContextProtocol.Server.RequestContext<ModelContextProtocol.Protocol.Types.CallToolRequestParams>, System.Threading.CancellationToken, System.Threading.Tasks.ValueTask<ModelContextProtocol.Protocol.Types.CallToolResponse>> handler = {Method = {System.Reflection.RuntimeMethodInfo}}, ModelContextProtocol.Protocol.Types.CallToolRequestParams args = {ModelContextProtocol.Protocol.Types.CallToolRequestParams}, System.Threading.CancellationToken cancellationToken = IsCancellationRequested = false)
 	ModelContextProtocol.dll!ModelContextProtocol.Server.McpServer.InvokeHandlerAsync<ModelContextProtocol.Protocol.Types.CallToolRequestParams, ModelContextProtocol.Protocol.Types.CallToolResponse>(System.Func<ModelContextProtocol.Server.RequestContext<ModelContextProtocol.Protocol.Types.CallToolRequestParams>, System.Threading.CancellationToken, System.Threading.Tasks.ValueTask<ModelContextProtocol.Protocol.Types.CallToolResponse>> handler = {Method = {System.Reflection.RuntimeMethodInfo}}, ModelContextProtocol.Protocol.Types.CallToolRequestParams args = {ModelContextProtocol.Protocol.Types.CallToolRequestParams}, ModelContextProtocol.Protocol.Transport.ITransport destinationTransport = null, System.Threading.CancellationToken cancellationToken = IsCancellationRequested = false)
 	ModelContextProtocol.dll!ModelContextProtocol.Server.McpServer.SetHandler.AnonymousMethod__0(ModelContextProtocol.Protocol.Types.CallToolRequestParams request = {ModelContextProtocol.Protocol.Types.CallToolRequestParams}, ModelContextProtocol.Protocol.Transport.ITransport destinationTransport = null, System.Threading.CancellationToken cancellationToken = IsCancellationRequested = false)
 	ModelContextProtocol.dll!ModelContextProtocol.Shared.RequestHandlers.Set.AnonymousMethod__0(ModelContextProtocol.Protocol.Messages.JsonRpcRequest request = {ModelContextProtocol.Protocol.Messages.JsonRpcRequest}, System.Threading.CancellationToken cancellationToken = IsCancellationRequested = false)
 	ModelContextProtocol.dll!ModelContextProtocol.Shared.McpSession.HandleRequest(ModelContextProtocol.Protocol.Messages.JsonRpcRequest request = {ModelContextProtocol.Protocol.Messages.JsonRpcRequest}, System.Threading.CancellationToken cancellationToken = IsCancellationRequested = false)
 	ModelContextProtocol.dll!ModelContextProtocol.Shared.McpSession.HandleMessageAsync(ModelContextProtocol.Protocol.Messages.JsonRpcMessage message = {ModelContextProtocol.Protocol.Messages.JsonRpcRequest}, System.Threading.CancellationToken cancellationToken = IsCancellationRequested = false)
 	ModelContextProtocol.dll!ModelContextProtocol.Shared.McpSession.ProcessMessagesAsync.__ProcessMessageAsync|0()
```

其消息入口代码是在 `McpSession.ProcessMessagesAsync` 方法里面，其逻辑如下

```csharp
internal sealed partial class McpSession
{
    public async Task ProcessMessagesAsync(CancellationToken cancellationToken)
    {
            await foreach (var message in _transport.MessageReader.ReadAllAsync(cancellationToken).ConfigureAwait(false))
            {
                ... // 调度收到的消息的逻辑
            }
    }
}
```

如以上代码所示，可见就是从传入的 ITransport 的 MessageReader 属性里面读取到客户端发送过来的输入数据，将其进行调度的

以上就是在相同的一个进程内进行 MCP 客户端和服务端通讯的简单示例代码

整个 Program.cs 代码如下

```csharp
// See https://aka.ms/new-console-template for more information

using System.Text.Json;
using System.Threading.Channels;
using ModelContextProtocol;
using ModelContextProtocol.Client;
using ModelContextProtocol.Protocol.Messages;
using ModelContextProtocol.Protocol.Transport;
using ModelContextProtocol.Protocol.Types;
using ModelContextProtocol.Server;

var transportFactory = new InterprocessTransportFactory();

McpServerOptions options = new()
{
    ServerInfo = new Implementation() { Name = "MyServer", Version = "1.0.0" },
    Capabilities = new ServerCapabilities()
    {
        Tools = new ToolsCapability()
        {
            ListToolsHandler = (request, cancellationToken) =>
                ValueTask.FromResult(new ListToolsResult()
                {
                    Tools =
                    [
                        new Tool()
                        {
                            Name = "echo",
                            Description = "Echoes the input back to the client.",
                            InputSchema = JsonSerializer.Deserialize<JsonElement>("""
                                {
                                    "type": "object",
                                    "properties": {
                                      "message": {
                                        "type": "string",
                                        "description": "The input to echo back"
                                      }
                                    },
                                    "required": ["message"]
                                }
                                """),
                        }
                    ]
                }),

            CallToolHandler = (request, cancellationToken) =>
            {
                if (request.Params?.Name == "echo")
                {
                    if (request.Params.Arguments?.TryGetValue("message", out var message) is not true)
                    {
                        throw new McpException("Missing required argument 'message'");
                    }

                    return ValueTask.FromResult(new CallToolResponse()
                    {
                        Content = [new Content() { Text = $"Echo: {message}", Type = "text" }]
                    });
                }

                throw new McpException($"Unknown tool: '{request.Params?.Name}'");
            },
        }
    },
};

await using IMcpServer server = McpServerFactory.Create(transportFactory.GetServerTransport(), options);
_ = server.RunAsync();

// 以下是客户端代码
var client = await McpClientFactory.CreateAsync(transportFactory.GetClientTransport());

// Print the list of tools available from the server.
foreach (var tool in await client.ListToolsAsync())
{
    Console.WriteLine($"{tool.Name} ({tool.Description})");
}

var dictionary = new Dictionary<string, object?>()
{
    { "message", "Hello, World!" }
};
CallToolResponse callToolResponse = await client.CallToolAsync("echo", dictionary);
foreach (var content in callToolResponse.Content)
{
    Console.WriteLine($"CallToolResponse: Type={content.Type} Text='{content.Text}'");
}

class InterprocessTransportFactory
{
    public InterprocessTransportFactory()
    {
        _clientTransport = new ClientTransport(this);
        _serverTransport = new ServerTransport(this);
    }

    private readonly ClientTransport _clientTransport;
    private readonly ServerTransport _serverTransport;

    public IClientTransport GetClientTransport()
        => _clientTransport;

    public ITransport GetServerTransport()
        => _serverTransport;

    class ClientTransport(InterprocessTransportFactory factory) : TransportBase, IClientTransport, ITransport
    {
        private readonly InterprocessTransportFactory _factory = factory;

        public Task<ITransport> ConnectAsync(CancellationToken cancellationToken = new CancellationToken())
        {
            return Task.FromResult<ITransport>(this);
        }

        public string Name => "ClientTransport";

        public override async Task SendMessageAsync(JsonRpcMessage message,
            CancellationToken cancellationToken = new CancellationToken())
        {
            await _factory._serverTransport.Channel.Writer.WriteAsync(message, cancellationToken);
        }
    }

    class ServerTransport(InterprocessTransportFactory factory) : TransportBase
    {
        private readonly InterprocessTransportFactory _factory = factory;

        public override async Task SendMessageAsync(JsonRpcMessage message,
            CancellationToken cancellationToken = new CancellationToken())
        {
            await _factory._clientTransport.Channel.Writer.WriteAsync(message, cancellationToken);
        }
    }

    abstract class TransportBase : ITransport
    {
        public Channel<JsonRpcMessage> Channel { get; } =
            System.Threading.Channels.Channel.CreateUnbounded<JsonRpcMessage>();

        public ValueTask DisposeAsync()
        {
            Channel.Writer.Complete();
            return ValueTask.CompletedTask;
        }

        public abstract Task SendMessageAsync(JsonRpcMessage message,
            CancellationToken cancellationToken = new CancellationToken());

        public ChannelReader<JsonRpcMessage> MessageReader => Channel.Reader;
    }
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/f4257182f3ab9e3e6deb81d41c5716d3a219e9a6/Workbench/FawheliraLemjawlelcalcelkener) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/f4257182f3ab9e3e6deb81d41c5716d3a219e9a6/Workbench/FawheliraLemjawlelcalcelkener) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f4257182f3ab9e3e6deb81d41c5716d3a219e9a6
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f4257182f3ab9e3e6deb81d41c5716d3a219e9a6
```

获取代码之后，进入 Workbench/FawheliraLemjawlelcalcelkener 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。