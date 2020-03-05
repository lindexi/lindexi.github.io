# asp dotnet core 简单开发P2P中央服务器

在做P2P的时候，如何让设备发现是整个开发里面最重要的部分。可以采用的方式有组播、扫描局域网、追踪服务器发现等方法。其中效率最高，发现效果最好的也就是使用中央服务器了。本文告诉大家如何使用 ASP.NET Core 写一个简单的 P2P 追踪服务器

<!--more-->
<!-- CreateTime:2019/11/1 19:40:33 -->

<!-- csdn -->

在 P2P 里面的追踪服务器最重要的功能就是告诉设备，他周围有哪些设备，或告诉设备他需要的资源在哪些设备。这里只是告诉设备周围有哪些设备的就是本文需要开发的服务器，而告诉资源的就是 BT 服务器做的事情。两个方法对应不同的业务

只是告知周围设备的，适合用来局域网连接上。通过客户端访问，可以知道所在的外网 IP 地址，将记录的相同的外网 IP 地址的客户端返回就完成了。对比资源方式的优点在于，现在很多 BT 服务器都因为访问量太大而难以使用，原因是客户端每个资源都需要在服务器端注册，假设有1w个客户端，而每个客户端有100个资源，假设每10分钟需要注册一次。也就是每10分钟有100w次访问。当然这样的效果也就是很好的，面向外网有大量的客户端，能返回资源在哪个客户端可以提高资源寻找速度

本文的服务器也就是拿到客户端访问的 IP 然后返回记录的相同的外网 IP 地址的客户端

也就是在客户端访问的时候，需要客户端将自己的内网 IP 告诉服务器端，这样服务器端就将这个内网 IP 记下。下次在相同局域网有另一个客户端访问就可以返回记录的内网地址

当然，如果需要支持外网也没问题，只需要将记录的所有客户端选取活跃返回就可以

打开 VisualStudio 2019 新建一个 asp dotnet core 项目

因为涉及到数据库的存储，需要存放客户端的 IP 地址和活跃时间。所以需要装上相关的库。我的设备十分强，有32G的内存，所以我就使用内存作为数据库，通过安装下面库使用 EF 协助

```csharp
    <PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="3.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="3.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="3.0.0">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
    <PackageReference Include="Microsoft.Extensions.Logging.Debug" Version="3.0.0" />
    <PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="3.0.0" />
```

现在打开 Startup.cs 修改使用内存作为数据库

```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();

            services.AddDbContext<NodeContext>(options =>
                    options.UseInMemoryDatabase("node"));
        }
```

然后创建 NodeContext 作为存放客户端的信息的数据

```csharp
    public class NodeContext : DbContext
    {
        public NodeContext (DbContextOptions<NodeContext> options)
            : base(options)
        {
        }

        public DbSet<KeahelnawwalyoNelwerchaje.Node> Node { get; set; }
    }

    public class Node
    {
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public int Id { set; get; }

        public string MainIp { set; get; }

        public string LocalIp { set; get; }

        public DateTime LastUpdate { set; get; }
    }
```

对于客户端需要存的信息是客户端的外网 IP 地址，也就是 MainIp 属性，客户端的内网 IP 地址，也就是 LocalIp 属性，还有客户端活跃时间

客户端的访问通过 get 的方法，在参数传入客户端内网 IP 地址

```csharp
    [Route("api/[controller]")]
    [ApiController]
    public class PeerController : ControllerBase
    {
        public PeerController(NodeContext context)
        {
            _context = context;
        }

        [HttpGet("{localIp}")]
        public IActionResult GetPeer(string localIp)
        {
        	// 忽略代码
        }

        private readonly NodeContext _context;

```

在 GetPeer 方法可以通过[asp dotnet core 获取用户真实 IP 地址](https://blog.lindexi.com/post/asp-dotnet-core-%E4%BB%8E-Frp-%E8%8E%B7%E5%8F%96%E7%94%A8%E6%88%B7%E7%9C%9F%E5%AE%9E-IP-%E5%9C%B0%E5%9D%80.html) 获取客户端的地址

从服务器找到相同的地址的客户端，更新当前客户端的时间，返回其他的客户端信息

```csharp
        [HttpGet("{localIp}")]
        public IActionResult GetPeer(string localIp)
        {
            var ip = GetIp();

            var nodeList = _context.Node.Where(temp => temp.MainIp == ip).ToList();

            var node = nodeList.FirstOrDefault(temp => temp.LocalIp == localIp);

            if (node != null)
            {
                _context.Node.Remove(node);
                nodeList.Remove(node);
            }

            _context.Node.Add(new Node
            {
                MainIp = ip,
                LocalIp = localIp,
                LastUpdate = DateTime.Now
            });

            _context.SaveChanges();
            return Ok(string.Join(';', nodeList.Select(temp => temp.LocalIp)));
        }
```

当然，有一些客户端可能很久没有连接，这些需要判断如果过了一段时间没有活跃就从数据库移除客户端

```csharp
            var removeList = new List<Node>();

            for (var i = 0; i < nodeList.Count; i++)
            {
                if (DateTime.Now - nodeList[i].LastUpdate > TimeSpan.FromHours(2))
                {
                    removeList.Add(nodeList[i]);
                    nodeList.RemoveAt(i);
                    i--;
                }
            }

            _context.Node.RemoveRange(removeList);
```

修改后的代码

```csharp
        [HttpGet("{localIp}")]
        public IActionResult GetPeer(string localIp)
        {
            var ip = GetIp();

            var nodeList = _context.Node.Where(temp => temp.MainIp == ip).ToList();

            var removeList = new List<Node>();

            for (var i = 0; i < nodeList.Count; i++)
            {
                if (DateTime.Now - nodeList[i].LastUpdate > TimeSpan.FromHours(2))
                {
                    removeList.Add(nodeList[i]);
                    nodeList.RemoveAt(i);
                    i--;
                }
            }

            var node = nodeList.FirstOrDefault(temp => temp.LocalIp == localIp);

            if (node != null)
            {
                _context.Node.Remove(node);
                nodeList.Remove(node);
            }

            _context.Node.Add(new Node
            {
                MainIp = ip,
                LocalIp = localIp,
                LastUpdate = DateTime.Now
            });
            _context.Node.RemoveRange(removeList);

            _context.SaveChanges();
            return Ok(string.Join(';', nodeList.Select(temp => temp.LocalIp)));
        }
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/blob/2392b76ca4cccae4df31938bb3583848ca041ac9/KeahelnawwalyoNelwerchaje/) 欢迎小伙伴访问

这样就完成了简单的追踪服务器，可以看到只需要很少的代码。客户端访问方法是通过 get 加上自己的内网地址，然后读取返回内容，用分号分开多个地址

为什么我不用 json 返回？原因是我的客户端都是很少的代码开发的，不想使用 json 库，有些客户端使用 c 写的，所以只能使用简单 get 方法，返回的也是字符串

有小伙伴问如果有一个外网地址就访问一次，那是不是数据库的内容就会占用。其实我不关注这个问题，因为我使用内存数据库，我大概几天就关机一次。另外，按照每个客户端报告一个内网 IP 加上端口，也就是大概21个字符，加上外网 IP 和 Id 这些属性，可以看到数据量是非常小。假设每个客户端需要 1kb 的内存，那么 1G 的内存足够 100w 客户端，如果有这么多客户端，我就可以去打广告。然而我只有 10 个客户端

本文的代码可以修改一下在你的项目中使用，非常简单，但是效果不错

客户端需要[获取本机 IP 地址](https://blog.lindexi.com/post/dotnet-%E8%8E%B7%E5%8F%96%E6%9C%AC%E6%9C%BA-IP-%E5%9C%B0%E5%9D%80%E6%96%B9%E6%B3%95.html) 加上本机的端口，拼接链接访问

```csharp
    var localIp = string.Join(';',
                   GetLocalIpList().Select(temp => $"{temp}:{port}"));
```

上面代码的 GetLocalIpList 方法请看我博客 [dotnet 获取本机 IP 地址方法](https://blog.lindexi.com/post/dotnet-%E8%8E%B7%E5%8F%96%E6%9C%AC%E6%9C%BA-IP-%E5%9C%B0%E5%9D%80%E6%96%B9%E6%B3%95.html) 

然后拼接链接

```csharp
        var url = $"http://p2p.api.acmx.xyz/api/peer/{localIp}";
```

上面的链接就是我部署的链接，如果小伙伴不想自己写服务器，也可以用我的。如果我关机了，这个链接就访问不到

我的服务器虽然很好，但是网很差，所以我设置了超时时间比较长

```csharp
                    var httpClient = new HttpClient()
                    {
                        Timeout = TimeSpan.FromMinutes(10)
                    };

                    using (httpClient)
                    {
                        var remoteIp = await httpClient.GetStringAsync(url);
                        var ipList = GetIpList(remoteIp).Where(temp =>
                            !string.IsNullOrEmpty(temp.ip) && !string.IsNullOrEmpty(temp.port)).ToList();
                    }
```

这里的 GetIpList 就是解析服务器返回

```csharp
        private IEnumerable<(string ip, string port)> GetIpList(string remoteIp)
        {
            var ipList = remoteIp.Split(';');
            foreach (var ip in ipList)
            {
                yield return IpRegex.Parse(ip);
            }
        }

    class IpRegex
    {
        public static (string ip, string port) Parse(string str)
        {
            var regex = new Regex(@"(\d+\.\d+\.\d+\.\d+):(\d+)");

            var match = regex.Match(str);

            if (match.Success)
            {
                var ip = match.Groups[1].Value;
                var port = match.Groups[2].Value;

                return (ip, port);
            }

            return default;
        }
    }
```

现在拿到了一些 IP 和端口，尝试访问这些客户端看能不能访问



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
