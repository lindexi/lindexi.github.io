# dotnet 教你写一个可以搞炸本机所有 WCF 应用的程序方法

作为团队里面挖掘机出身的我，怎么能不多挖一些坑好将小伙伴们都埋进去呢。本文来告诉大家一个有趣且简单的方法，此方法可以将本机的 WCF 玩坏，不敢说真的搞炸本机所有 WCF 应用，但搞炸大部分基于 WCF 的软件还是没有问题的。阅读本文，你可以不仅可以了解到有这样的逗比方法，更重要的是在你的 WCF 模块炸掉的时候，你知道要甩锅给谁

<!--more-->
<!-- 博客 -->

本文如此逗比的方法是由 [lsj](https://blog.sdlsj.net) 小伙伴发现的，但是他不想记录如此逗比的方法，于是就交给我来水了

在开始之前，咱先来复习如何制作一个简单的 WCF 服务端和客户端的方法。用不着官方文档提供的十分繁琐的方式，咱直接明了，通过简单的控制台，利用WCF实现本机 IPC 进程间通讯

咱将先制作一个简单的 WCF 进程间通讯的服务端和客户端两个控制台项目，用来演示在管道下的 WCF 应用的运行情况。接着再添加一个用来捣乱的 WCF 服务器端的控制台项目，让这个项目影响到原有工作的好好的演示项目

当前是 2021.08.22 社区版本发布了 WCF Core 的 0.2.0 版本，功能上还没有追平 .NET Framework 的版本，因此本文依然使用 .NET Framework 版本的 WCF 进行演示

先来演示的 WCF 服务端的控制台应用，咱通过 .NET 5 创建出项目，接着编辑 csproj 文件，将 net5.0 修改为 net45 从而返回到 .NET Framework 版本。服务端的 csproj 文件代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net45</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <Reference Include="System.ServiceModel" />
  </ItemGroup>

</Project>
```

使用 .NET 5 创建项目的优势是新建出来的项目是 SDK 风格的，方便更改。为了使用上 WCF 在 csproj 上添加 `System.ServiceModel` 的引用

在 SDK Style 的 csproj 项目文件上，添加对 WCF 引用的方法是在 csproj 上添加如下代码

```xml
  <ItemGroup>
    <Reference Include="System.ServiceModel" />
  </ItemGroup>
```

按照惯例，定义类型，此类型将包含一个类和一个接口。类是在服务端运行的，而接口是给客户端使用的。这部分基础知识不在本文描述，更多基础知识请参阅本文最后的由换头像大大编写的入门博客

```csharp
    [ServiceBehavior(InstanceContextMode = InstanceContextMode.Single, ConcurrencyMode = ConcurrencyMode.Single)]
    public class DataService: IDataServer
    {
        public void Foo(string name)
        {
            Console.WriteLine(name);
        }
    }

    [ServiceContract]
    public interface IDataServer
    {
        [OperationContract]
        void Foo(string name);
    }
```

接着打开 Program.cs 文件，在此文件编写入口函数，在入口函数启动服务，如以下代码

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            var dataService = new DataService();
            Uri address = new Uri("net.pipe://localhost/MyWCFConnection");
            using (ServiceHost host = new ServiceHost(dataService, address))
            {
                host.Open();

                while (true)
                {
                    Thread.Sleep(100);
                }
            }


        }
    }
```

上面代码使用了 `net.pipe://localhost/MyWCFConnection` 启动了使用管道的 WCF 服务

接着采用相同的方法，也是使用 .NET 5 创建控制台，修改为 .NET Framework 版本的客户端控制台

在客户端控制台的 csproj 文件代码和服务端的相同。放心，在本文最后有所有的源代码，部分细节还请忽略。在客户端里面，添加上了刚才定义的 IDataServer 接口，抄代码即可

在客户端的入口添加如下代码，用于连上服务端，然后远程调用服务端的某个方法

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            Uri address = new Uri("net.pipe://localhost/MyWCFConnection");

            var dataServer = ChannelFactory<IDataServer>.CreateChannel(new NetNamedPipeBinding(),new EndpointAddress(address));
            dataServer.Foo("123");
        }
    }
```

先启动服务端，再启动客户端。预期是服务端的 DataServer 的 Foo 方法将会被客户端进行调用，被客户端传入了 `"123"` 在服务端的控制台输出

接下来开始开发一个用来捣乱的 WCF 控制台，这是一个 WCF 服务端。这个控制台应用的 csproj 和上面两个相同，唯一不同的是在入口程序的定义和运行的方式。在入口里面使用如下代码启动服务

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            var dataService = new DataService();
            Uri address = new Uri("net.pipe://localhost/");
            using (ServiceHost host = new ServiceHost(dataService, address))
            {
                host.Open();

                while (true)
                {
                    Thread.Sleep(100);
                }
            }
        }
    }
```

可以看到，以上的写法是不被推荐的，采用了不加上具体的逻辑的管道

- `net.pipe://localhost/MyWCFConnection` 这是通用的方式
- `net.pipe://localhost/` 这是不符合约定的

接着构建出这个捣乱的应用，使用管理员权限打开他。然后再尝试启动原本可以好好干活的演示应用，可以看到演示应用的客户端炸掉了，提示如下

```
System.ServiceModel.EndpointNotFoundException:“由于 AddressFilter 在 EndpointDispatcher 不匹配，To 为“net.pipe://localhost/MyWCFConnection”的消息无法在接收方处理。请检查发送方和接收方的 EndpointAddresses 是否一致。”
```

以上的错误提示和服务端 WCF 没有启动或者在客户端配置的连接字符串和服务端配置的不相同的是一样的提示方式

原因其实比较复杂一点，简单说就是 WCF 的连接字符串，在通过管道的方式的时候，不是直接作为管道名的。而是将此连接字符串映射到某个共享内存里面，在共享内存里面存放实际的管道名。而上面用来捣乱的应用就是用了不符合约定的方式，让客户端在尝试发现服务端的时候，先碰到了捣乱的应用，又因为权限不足从而失败。如果此时将演示用的服务端也采用管理员权限运行，而演示用的客户端依然是非管理员权限运行，那么演示程序还能正常工作

想要写一个用来搞炸本机大部分的基于 WCF 做 IPC 进程间通讯的捣乱应用，只需要设置 WCF 连接字符串为 `net.pipe://localhost/` 接着使用管理员运行即可，如运行为服务

这个问题其实是某个用户报告给我的，经过了 [lsj](https://blog.sdlsj.net) 使用了各个黑科技的方式调试，加上堆栈网大佬们的回复，了解到了是 `DropboxOEM.exe` 服务挖的坑。然而除此之外，在堆栈网上面也列出了其他的很多应用也会导致此问题。这个问题其实 WCF 和应用两边都有锅

在 WCF 上，为了安全考虑，反而挖了如此的坑，会让应用受到了本机内其他在运行的应用的影响。另一方面，其实 WCF 也算背锅，因为如果应用乱来，导致影响其他应用，似乎在 Win32 设计层面本身就有这样的问题，如应用自己去删掉了某个系统关键文件等。只是 WCF 这个锅不好定位在于，使用 WCF 不属于唯一方式，这就意味着其他的 IPC 也许能活，给用户的感觉就是为什么我其他的应用都能工作好好的，就你的应用炸了

另外，我还测试了其他的组合：

- 演示程序的 WCF 连接字符串： `net.pipe://127.0.0.1/MyWCFConnection`
- 捣乱程序 WCF 连接字符串: `"net.pipe://localhost/"`
- 捣乱程序使用管理员权限运行

结论：炸

后续为了升级到 .NET Core 或 .NET 5 等更高版本的 .NET 我开源了一个追求稳定的 IPC 库，请看 [dotnet-campus/dotnetCampus.Ipc: 本机内多进程通讯库](https://github.com/dotnet-campus/dotnetCampus.Ipc/)

当前此开源库还没有实际落地，缺乏大量的诡异的用户环境的适配。预计大概到 2022 的时候，这个库能更加稳定

本文所有代码放在[github](https://github.com/lindexi/lindexi_gd/tree/26aa3294d0bc40ba7e312891c958fa170c3d51f0/ChigiwejefiKemhakerhawee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/26aa3294d0bc40ba7e312891c958fa170c3d51f0/ChigiwejefiKemhakerhawee) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 26aa3294d0bc40ba7e312891c958fa170c3d51f0
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 ChigiwejefiKemhakerhawee 文件夹

此问题也能在堆栈网找到，请看 [.net - 3rd party app breaks our WCF application - Stack Overflow](https://stackoverflow.com/questions/15981392/3rd-party-app-breaks-our-wcf-application/15987095)


更多 WCF 请参阅：

- [wcf入门（1） - huangtengxiao](https://huangtengxiao.gitee.io/post/wcf%E5%85%A5%E9%97%A8-1.html)
- [wcf入门（2） - huangtengxiao](https://huangtengxiao.gitee.io/post/wcf%E5%85%A5%E9%97%A8-2.html)
- [wcf入门（3） - huangtengxiao](https://huangtengxiao.gitee.io/post/wcf%E5%85%A5%E9%97%A8-3.html)
- [wcf入门（4） - huangtengxiao](https://huangtengxiao.gitee.io/post/wcf%E5%85%A5%E9%97%A8-4.html)
- [wcf入门（5） - huangtengxiao](https://huangtengxiao.gitee.io/post/wcf%E5%85%A5%E9%97%A8-5.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
