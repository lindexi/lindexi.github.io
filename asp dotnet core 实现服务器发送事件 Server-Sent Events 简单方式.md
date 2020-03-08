# asp dotnet core 实现服务器发送事件 Server-Sent Events 简单方式

在客户端开发时可以通过轮询的方式拿到服务器端的数据，同时在客户端开发时，如果是将客户端也作为服务器端，那么之间的通讯将会十分简单。有个逗比的小伙伴想要用我的客户端魔改，但是他又不想学习什么知识，此时他需要拿到我客户端的实时信息，好在他知道一点 html 的知识，于是让我通过服务器发送事件 Server-Sent Events 而他写一个简陋的 html 去拿到我客户端的数据

这是一个简陋的开发端的工具，开源的好处就是，你觉得不爽，自己改哇。自己改不动就等开发者下班协助啦，本文就使用一个简单的方式在 asp dotnet core 实现服务器发送事件。虽然标题是 asp dotnet core 而实际上我的软件是一个桌面端软件

<!--more-->
<!-- CreateTime:2020/1/6 18:17:58 -->

<!-- csdn -->

其实服务器发送事件 Server-Sent Events 原理就是在请求发送的 stream 设置 `Content-Type` 为 `text/event-stream` 然后不断写入数据就可以了

新建一个控制器，在控制台里面的调用方法，注意需要是 get 方法哦（其实没有限制）这样写起来才简单

```csharp
    [ApiController]
    [Route("[controller]")]
    public class FooController : ControllerBase
    {
        [HttpGet]
        public async Task Get()
        {

        }
```

通过 Response 属性可以拿到请求，在 Header 添加 `Content-Type` 这样就可以告诉调用者返回的是服务器发送事件

```csharp
            var response = Response;
            response.Headers.Add("Content-Type", "text/event-stream");
```

不断给调用者发送数据的方法就是不断写入数据

```csharp
                await response
                    .WriteAsync($"data: 当前时间 {DateTime.Now} {Thread.CurrentThread.ManagedThreadId}\r\r");
```

注意写入数据的格式，要求是 `data: 信息 \r\r` 如果格式不对，那么调用方是接收不到数据

写入数据之后需要发送

```csharp
                response.Body.Flush();
```

如不断告诉调用方更新时间可以这样写

```csharp
        [HttpGet]
        public async Task Get()
        {
            var response = Response;
            response.Headers.Add("Content-Type", "text/event-stream");

            for (var i = 0; ; ++i)
            {
                await response
                    .WriteAsync($"data: 当前时间 {DateTime.Now} {Thread.CurrentThread.ManagedThreadId}\r\r");

                response.Body.Flush();
                await Task.Delay(TimeSpan.FromSeconds(1));
            }
        }
```

上面的代码请不要放在实际项目，因为方法被调用就不会停下

此时调用者就可以使用简单的方法拿到服务器发送的数据

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/blob/aa26d45303346118d72ef8560d295cc3f38f9809/LibageadairJayfufearker/) 下载代码运行，访问 `http://localhost:端口/index.html` 就可以看到网页不断刷新时间

当然更好的通讯方法是通过 Pipe 的方式通讯，可选框架是 WCF 等

如果是现代的开发，建议使用 SignalR 的方法发送数据，其实 SignalR 底层传输是 Web Socket, Server Sent Events 和 Long Polling 方法

[.net core HTML5支持服务器发送事件(Server-Sent Events)-单向消息传递数据推送（C#示例）](https://blog.csdn.net/qq_36577699/article/details/82627925)

本文开始标题是 WPF 发送Server-Sent Events给其他进程，但是实际上没有用到 WPF 的任何内容，于是修改了标题。如果你是因为工作需要用到这个技术，搜到本文，如果还有精力，那么我推荐你看一下 WCF 或 SignalR 的方法。如果没有，那么本文的代码也请不要抄，因为上面的代码会让方法不断运行

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
