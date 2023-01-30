
# dotnet 6 推荐一个可代替 .NET Remoting 的 IPC 库

本文将来和大家推荐一个基于最友好 MIT 协议的完全在 GitHub 上开源的，可代替 .NET Remoting 的 IPC 本机多进程通讯库

<!--more-->


<!-- CreateTime:2022/6/20 8:03:37 -->

<!-- 发布 -->

本机内多进程通讯 IPC 不同于跨设备系统的 RPC 通讯方式，大多数的 IPC 通讯都需要处理复杂的用户端环境问题。对于 RPC 通讯来说，大部分时候，服务端都在开发者完全管控的环境下运行。但 IPC 通讯则无论是服务端还是客户端都可能是在用户端运行的。然而用户端上，无论是系统还是其他环境都是十分复杂的，特别是在国内的，魔改的系统，凶狠的杀毒软件，这些都会让 IPC 通讯受到非预期的打断

传统的 dotnet 系的 IPC 手段有很多个，提供给开发使用的顶层框架也有很多，如 .NET Remoting 和 WCF 等。但是在迁移到 dotnet core 时，由于底层运行时机制的变更，如透明代理不再支持类对象只能支持接口的行为变更，就让 .NET Remoting 从机制性不受支持。为了方便将应用迁移到 dotnet core 框架上，可采用 dotnet campus 组织基于最友好的 MIT 协议开源的 [dotnetCampus.Ipc 开源库](https://github.com/dotnet-campus/dotnetCampus.Ipc)进行本机内多进程通讯

此 [dotnetCampus.Ipc](https://github.com/dotnet-campus/dotnetCampus.Ipc) 开源库底层可基于命名管道进行通讯，经过了约千万台设备近一年的测试，发现通过此方式的通讯稳定性极高。开源仓库地址：[https://github.com/dotnet-campus/dotnetCampus.Ipc](https://github.com/dotnet-campus/dotnetCampus.Ipc)

本文将告诉大家如何使用 [dotnetCampus.Ipc](https://github.com/dotnet-campus/dotnetCampus.Ipc) 库实现类似 .NET Remoting 的 IPC 通讯效果。无缝替换 .NET Remoting 是做不到的，需要做一定的迁移，还请先看一下本 IPC 是如何使用的

先新建两个项目，分别是 IpcRemotingObjectServerDemo 作为服务端，和 IpcRemotingObjectClientDemo 作为客户端。本质上来说 [dotnetCampus.Ipc](https://github.com/dotnet-campus/dotnetCampus.Ipc) 库是采用 P2P 模型，没有服务端和客户端之分，只是本文为了对标 .NET Remoting 而强行划分

按照惯例，先通过 NuGet 给 IpcRemotingObjectServerDemo 和 IpcRemotingObjectClientDemo 安装 dotnetCampus.Ipc 库。如上文，由于 dotnetCampus.Ipc 采用 P2P 模型，没有客户端和服务端之分，因此作为客户端和作为服务端的项目安装的 NuGet 包都是相同的

先定义公共约束逻辑，也就是定义的接口。由于 .NET Core 变更了底层透明代理的机制，不再允许透明代理类型，只能透明代理接口。因此本 IPC 库自然比 .NET Remoting 弱化一些，只能定义接口，再定义实现逻辑，而不能和 .NET Remoting 一样，将接口约束和具体实现都在一个类里面实现

```csharp
interface IFoo
{
    int Add(int a, int b);

    Task<string> AddAsync(string a, int b);
}
```

本 IPC 库在当前 2020.06 只支持属性和方法的远程过程调用方式，不支持事件和委托。好在事件和委托可以采用反向方法调用间接实现，也就是原本是需要做事件的逻辑，换成调用对方的一个方法的方式。也好在原本在 .NET Remoting 上，委托和事件是比较难使用的，导致了更换起来还是比较清真的

为了和 IPC 库对接，给这个接口标记 IpcPublicAttribute 特性，在特性上面可选带上参数，如下面代码带上了 IgnoresIpcException 表示忽略 IPC 连接和 IPC 通讯本身的异常，但不忽略业务端抛出的异常，再添加 Timeout 表示方法或属性调用的超时时间

```csharp
using dotnetCampus.Ipc.CompilerServices.Attributes;

[IpcPublic(IgnoresIpcException = true, Timeout = 1000)]
interface IFoo
{
    int Add(int a, int b);

    Task<string> AddAsync(string a, int b);
}
```

接着编写服务器端的实现逻辑

```csharp
class Foo : IFoo
{
    public int Add(int a, int b)
    {
        Console.WriteLine($"a({a})+b({b})={a + b}");
        return a + b;
    }

    public async Task<string> AddAsync(string a, int b)
    {
        return await Task.Run(() =>
        {
            Console.WriteLine($"a({a})+b({b})={a + b}");
            return a + b;
        });
    }
}
```

在服务端编写启动 IPC 的和注册 IFoo 服务的逻辑

```csharp
using dotnetCampus.Ipc.CompilerServices.GeneratedProxies;
using dotnetCampus.Ipc.Pipes;
using IpcRemotingObjectServerDemo;

var ipcProvider = new IpcProvider("IpcRemotingObjectServerDemo");

ipcProvider.CreateIpcJoint<IFoo>(new Foo());
ipcProvider.StartServer();

Console.Read();
```

在本 IPC 库里，初始化时可以传入 IPC 名，如以上创建名为 `IpcRemotingObjectServerDemo` 的 IPC 服务。其他的端即可通过此名称连接上这个 IPC 服务。再接着是通过 `CreateIpcJoint` 方法注册 IPC 服务

完成注册之后，通过 StartServer 方法开启服务

为了不让主方法退出，导致进程退出，再加上 `Console.Read` 代码。如此就完成了服务端代码的编写

接下来就是客户端的代码了

需要在客户端定义一个一模一样的接口，当然，更好的方法就是在 VisualStudio 里面添加现有项，将在服务端定义的接口文件作为链接引用进来。在本 IPC 的接口要求是要求接口和命名空间都是一模一样的

定义完成接口之后，就可以开始编写客户端的 IPC 连接逻辑，代码如下。通过以下代码即可建立和服务端的连接

```csharp
var ipcProvider = new IpcProvider("IpcRemotingObjectClientDemo");

ipcProvider.StartServer();

PeerProxy peer = await ipcProvider.GetAndConnectToPeerAsync("IpcRemotingObjectServerDemo");
```

以上的 peer 对象就是用来代表服务端的，通过调用 CreateIpcProxy 传入此 peer 对象即可拿到服务端注册的远程 Foo 实例的代理

```csharp
IFoo foo = ipcProvider.CreateIpcProxy<IFoo>(peer);
```

接着即可调用此接口对应的方法，代码如下

```csharp
Console.WriteLine(await foo.AddAsync("a", 1));
Console.WriteLine(foo.Add(1, 2));
```

除了定义和连接部分和 .NET Remoting 有比较大的差别之外，使用方基本上没有多少差别

这就是 [dotnetCampus.Ipc](https://github.com/dotnet-campus/dotnetCampus.Ipc) 库的远程对象调用的使用方法，例子代码放在代码仓库 [https://github.com/dotnet-campus/dotnetCampus.Ipc](https://github.com/dotnet-campus/dotnetCampus.Ipc) 里





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。