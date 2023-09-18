
# dotnet 推荐一个使用 Json 直接路由通讯的 IPC 库

本文将和大家推荐一个我所在团队开源的本机多进程通讯 IPC 库，此 IPC 支持使用 JSON 格式进行直接路由通讯，具有使用方便，稳定性高，性能好的优点

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

这是我所在的团队在 GitHub 上使用最友好的 MIT 协议完全开源的项目：[https://github.com/dotnet-campus/dotnetCampus.Ipc](https://github.com/dotnet-campus/dotnetCampus.Ipc)

这个开源项目开源的是一个 IPC 库，而本文将要介绍的是这个 IPC 库其中的一个功能：使用 Json 格式进行直接路由通讯

接下来我将一步步告诉大家如何使用这个功能实现本机多进程通讯

按照 dotnet 的惯例，先安装 NuGet 库。可以通过右击项目进入 NuGet 管理器，搜 [dotnetCampus.Ipc](https://www.nuget.org/packages/dotnetCampus.Ipc) 进行安装，也可以编辑 csproj 项目文件进行快速安装，如添加以下代码到你的 csproj 项目文件里面

```xml
<PackageReference Include="dotnetCampus.Ipc" Version="2.0.0-alpha405" />
```

原本的这个 IPC 库设计上是 P2P 的方式，也就是没有明确的客户端和服务端。只是在使用直接路由方式进行通讯的时候，在概念上有服务端和客户端。在本文接下来的描述里面会明确说明客户端和服务端的逻辑，但底层上依然是 P2P 模式，不存在让某个进程作为本质的服务端或客户端，每个进程都可以既是客户端又是服务端

先编写创建服务端的代码逻辑。服务端需要给一个服务名，客户端可以通过此服务名连接上服务端，从设计上只允许客户端主动连接服务端。服务名从底层上将会作为管道名，请使用符合管道命名规则的方式进行命名，一般都是英文字符大小写加上数字下划线

```csharp
            // 初始化服务端
            var serverName = "JsonIpcDirectRoutedProviderTest_Request_1";
            var serverProvider = new JsonIpcDirectRoutedProvider(serverName);
```

这个 JsonIpcDirectRoutedProvider 类型只是一个 JSON 直接路由的 IPC 提供器，构造函数可选传入 IPC 配置，比如配置注入日志设置日志输出等级

在服务端上可以定义响应和通知的处理逻辑，以下代码定义的是对名为 “Foo1” 的直接路由的请求的处理逻辑

```csharp
            serverProvider.AddRequestHandler("Foo1", (FakeArgument arg) =>
            {
                return new FakeResult("Ok");
            });
```

请求处理也就是客户端发起一个请求给到服务端进行处理，服务端处理完成之后返回响应内容给到客户端。以上的 FakeArgument 类型参数就是由客户端发送给到服务端的参数内容，客户端发送到服务端的参数内容将会见过 Json 的序列化和反序列化的过程。接下来服务端的 FakeResult 返回值将作为响应，通过 IPC 传输给到客户端，也会经过序列化和反序列化的过程

服务端定义通知的处理逻辑例子如下，通知只有从客户端发过来的参数，不需要返回任何对象给到客户端，即客户端只是发过来一条通知给到服务端

```csharp
            var routedPath = "FooPath";
            serverProvider.AddNotifyHandler(routedPath, (FakeArgument arg) =>
            {
            });
```

同理，通知的 FakeArgument 参数也会经过序列化和反序列化的过程

服务端完成了路由事件的定义之后，即可通过 StartServer 方法进行启动

```csharp
            serverProvider.StartServer();
```

从 IPC 的设计上，要求在 StartServer 启动服务之前完成所有对路由事件的定义。在 StartServer 之后，禁止再 AddRequestHandler 或 AddNotifyHandler 添加处理逻辑。此设计是为了保证消息不丢失，防止存在消息在路由事件定义完成之前收到而丢失

以上连在一起的服务端的定义和启动代码如下

```csharp
            // 初始化服务端
            var serverName = "JsonIpcDirectRoutedProviderTest_Request_1";
            var serverProvider = new JsonIpcDirectRoutedProvider(serverName);

            serverProvider.AddRequestHandler("Foo1", (FakeArgument arg) =>
            {
                return new FakeResult("Ok");
            });

            serverProvider.AddRequestHandler("Foo2", (FakeArgument2 arg) =>
            {
                return new FakeResult2("Ok");
            });

            serverProvider.AddRequestHandler("Foo3", (FakeArgument3 arg) =>
            {
                return new FakeResult3("Ok");
            });

            var routedPath = "FooPath";
            serverProvider.AddNotifyHandler(routedPath, (FakeArgument arg) =>
            {
            });

            serverProvider.AddNotifyHandler("FooPath1", (FakeArgument1 arg) =>
            {
            });

            serverProvider.AddNotifyHandler("FooPath2", (FakeArgument2 arg) =>
            {
            });

            serverProvider.StartServer();
```

从业务实现上，可以将注入处理的逻辑分到多个不同的模块里面，通过传入 JsonIpcDirectRoutedProvider 给到多个模块，让多个模块在依赖注册阶段进行添加处理。完成之后再调用 StartServer 启动服务

以上完成了服务端部分代码的编写，接下来看看客户端部分的代码编写

本质上的 JsonIpcDirectRouted 依然是 P2P 的方式，而不是 客户端-服务端 的方式。客户端的创建也需要从 JsonIpcDirectRoutedProvider 获取到

```csharp
            // 创建客户端
            // 允许无参数，如果只是做客户端使用的话
            JsonIpcDirectRoutedProvider clientProvider = new();
            // 对于 clientProvider 来说，可选调用 StartServer 方法
            var clientProxy = await clientProvider.GetAndConnectClientAsync(serverName);
```

由于客户端不需要被服务端连接，客户端可以省略构造函数的参数

获取客户端时，需要调用 GetAndConnectClientAsync 方法传入服务端的服务名。如果此时的服务端还没启动，将会在 await 里面异步等待服务端启动且连接上服务端

获取到客户端对象之后，即可对服务器发起请求获取响应，也可以单向给服务端发送通知。以下是对服务端发起请求获取响应的例子

```csharp
            var argument = new FakeArgument("TestName", 1);
            FakeResult result = await clientProxy.GetResponseAsync<FakeResult>("Foo1", argument);
```

以上代码的 GetResponseAsync 第一个参数表示的是所请求的路由地址，第二个参数是一个对象，将会被 Json 序列化然后发送给服务端。返回值的 FakeResult 是服务端处理的返回值

以下是发送通知给服务端的例子

```csharp
            var argument = new FakeArgument("TestName", 1);
            await clientProxy.NotifyAsync("FooPath", argument);
```

发送通知时 await 返回只代表服务端收到了通知，不代表服务端处理通知完成

连在一起的客户端创建和通讯的代码如下

```csharp
            // 创建客户端
            // 允许无参数，如果只是做客户端使用的话
            JsonIpcDirectRoutedProvider clientProvider = new();
            // 对于 clientProvider 来说，可选调用 StartServer 方法
            var clientProxy = await clientProvider.GetAndConnectClientAsync(serverName);

            var result = await clientProxy.GetResponseAsync<FakeResult>("Foo1", argument);

            await clientProxy.NotifyAsync("Foo1", argument);
```

以上就是此 IPC 库的使用 Json 直接路由事件方式进行通讯的简单例子

此 IPC 库不仅提供了本文介绍的 Json 直接路由通讯方式，还可以使用 IPC 对象通讯方式，就如同 .NET Remoting 的用法差不多，详细请看 [dotnet 6 推荐一个可代替 .NET Remoting 的 IPC 库](https://blog.lindexi.com/post/dotnet-6-%E6%8E%A8%E8%8D%90%E4%B8%80%E4%B8%AA%E5%8F%AF%E4%BB%A3%E6%9B%BF-.NET-Remoting-%E7%9A%84-IPC-%E5%BA%93.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。