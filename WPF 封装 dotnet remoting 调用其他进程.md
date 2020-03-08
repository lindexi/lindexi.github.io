# WPF 封装 dotnet remoting 调用其他进程

本文告诉大家一个封装好的库，使用这个库可以快速搭建多进程相互使用。

<!--more-->
<!-- CreateTime:2019/2/16 9:40:26 -->

<!-- csdn -->
<div id="toc"></div>
<!-- 标签：WPF，dotnetremoting，rpc， -->

在 [WPF 使用RPC调用其他进程](https://lindexi.gitee.io/post/WPF-%E4%BD%BF%E7%94%A8RPC%E8%B0%83%E7%94%A8%E5%85%B6%E4%BB%96%E8%BF%9B%E7%A8%8B.html ) 已经告诉大家调用的原理，但是大家可以看到，如果自己写一个框架是比较难的。

因为我经常调用 C++ 代码，如果C++出现异常，软件会直接退出，所以我就想把 C++ 代码放在其他进程，这样C++运行出现异常了，软件也不会直接退出。

但是如果每次都需要自己写相同的代码，我是不同意的，因为很容易写错。

因为我的代码放在了公司代码使用，所以我不会把源代码放出来，但是大家可以通过复制本文的类来创建框架。

## 创建端口

创建端口包含一个接口和一个类，因为我需要在一个设备运行，所以为了性能，我不使用 http 连接，这时的端口可以使用一个字符串

为了区分两个程序，我把程序分为两个，一个是 WPF 程序，一个是C++程序。因为另一个程序主要是运行 C++ 代码。

为了让两个程序能联系，就需要约定端口，因为这是框架，可能需要使用 http 通信，所以就需要写一个接口，如果需要使用 http 修改端口就继承，这样框架才可以在很多地方使用。

```csharp
    /// <summary>
    ///     创建端口
    /// </summary>
    public interface IPortGenerator
    {
        /// <summary>
        ///     获得端口
        /// </summary>
        /// <returns></returns>
        string GetPort();
    }
```

因为我需要在一个系统运行两个程序，所以我的端口是这样写

```csharp
    /// <summary>
    ///     创建端口
    /// </summary>
    public class PortGenerator : IPortGenerator
    {
        /// <summary>
        ///     获得一个随机端口
        /// </summary>
        /// <returns></returns>
        public string GetPort()
        {
            const int maxPort = 65535;
            const int minPort = 49152;
            return (Process.GetCurrentProcess().Id % (maxPort - minPort) + minPort).ToString(); //获得进程ID作为端口号
        }
    }
```

## 调用软件

从 WPF 程序调用 C++ 程序需要告诉他参数，参数就是刚才的端口

这时 C++ 程序使用命令行解析，请看[安利一款非常好用的命令行参数库：McMaster.Extensions.CommandLineUtils - walterlv](https://walterlv.github.io/post/mcmaster-extensions-commandlineutils.html )

创建一个类用来解析

```csharp
    public class Options
    {
        [Option('p', "port", Required = true, HelpText = "远程开启的端口")]
        public string Port { get; set; }
    }
```

解析只需要使用主函数传入的 args 就可以拿到端口

```csharp
            Parser.Default.ParseArguments<Options>(args).MapResult(options =>
            {
                Console.WriteLine("端口号" + options.Port);
                new Thread(() =>
                {
                    Console.WriteLine("启动库");
                    Run(options.Port, assembly);
                    Console.WriteLine("启动完成");
                }).Start();
                return 0;
            }, _ => -1);
```

这里启动一个新的线程因为C++程序需要使用另一个线程去计算，主函数的线程会如果没有使用 `Console.Read()` 会退出。

现在 WPF 可以开始调用 C++ 程序，使用下面的代码进行管理

```csharp
    /// <summary>
    ///     管理其他进程
    /// </summary>
    public class RemoteProcessManager
    {
        /// <summary>
        ///     管理其他进程
        /// </summary>
        /// <param name="processName">进程名，用于启动进程</param>
        public RemoteProcessManager(string processName)
        {
            ProcessName = processName;
        }

        /// <summary>
        ///     获取管理的进程
        /// </summary>
        public string ProcessName { get; }

        /// <summary>
        ///     获取是否连接
        /// </summary>
        public bool IsConnected { get; private set; }

        /// <summary>
        ///     创建端口
        /// </summary>
        public IPortGenerator PortGenerator { get; set; }


        /// <summary>
        ///     远程应用退出时，建议监听后使用异常
        /// </summary>
        public event EventHandler RemoteExited;

        /// <summary>
        ///     连接
        /// </summary>
        public void Connect()
        {
            if (IsConnected)
            {
                throw new InvalidOperationException("禁止多次连接");
            }

            IsConnected = true;

            if (PortGenerator == null)
            {
                PortGenerator = new PortGenerator();
            }

            Port = PortGenerator.GetPort();

            //启动远程进程
            var st = new ProcessStartInfo(ProcessName, "-p " + Port);

#if !DEBUG
            st.CreateNoWindow = true;
            st.WindowStyle = ProcessWindowStyle.Hidden;
#endif
            var remoteGuardian = Process.Start(st); //监控远程应用

            if (remoteGuardian == null)
            {
                throw new RemoteProcessStartException("启动时出现返回值为空")
                {
                    Data = 
                    {
                        {"ProcessStartInfo", st}
                    }
                };
            }

            ProcessId = remoteGuardian.Id;
            remoteGuardian.EnableRaisingEvents = true;
            remoteGuardian.Exited += RemoteGuardian_OnExited;

            CleanRegister();

            _channel = Terminal.CreatChannel(); //客户端

            ChannelServices.RegisterChannel(_channel, false);
        }

        /// <summary>
        ///     从远程获得实例
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <returns></returns>
        public T GetObject<T>()
        {
            CheckProcess();
            return (T) Activator.GetObject(typeof(T),
                "Ipc://" + Port + "/" + typeof(T).Name);
        }

        /// <summary>
        ///     结束远程进程
        /// </summary>
        public void ExitProcess()
        {
            _isManualExit = true;

            var remoteProcessBrake = GetObject<RemoteProcessBrake>();
#pragma warning disable 618
            remoteProcessBrake.OnExit();
#pragma warning restore 618
        }

        private IChannel _channel;

        /// <summary>
        ///     是否手动退出远程应用
        /// </summary>
        private bool _isManualExit;

        private void RemoteGuardian_OnExited(object sender, EventArgs e)
        {
            IsConnected = false;
            ProcessId = -1;

            //手动退出就不需要事件
            if (_isManualExit)
            {
                return;
            }

            //防止360等垃圾软件觉得这个应用可以退出
            RemoteExited?.Invoke(sender, e);

            //即使被你退出了，我还是要启动，但是可能存在一些地方使用的变量放在本地，所以拿到的值就是之前的应用，还是需要用户重启
            Connect();
        }

        /// <summary>
        ///     清理注册，因为一个信道只能注册
        /// </summary>
        private void CleanRegister()
        {
            if (_channel != null)
            {
                ChannelServices.UnregisterChannel(_channel);
            }
        }


        private int ProcessId { set; get; } = -1;
        private string Port { get; set; }

        private void CheckProcess()
        {
            if (!IsConnected || ProcessId == -1)
            {
                throw new NativeProcessException("远程应用已经意外结束或没有启动");
            }

            var processes = Process.GetProcesses();
            if (ProcessId != -1 && processes.All(temp => temp.Id != ProcessId))
            {
#if DEBUG
                throw new NativeProcessException();
#else
                IsConnected = false;
                Connect();
#endif
            }
        }
    }

```

注意现在的代码存在很多类没有引用

从上面代码可以看到，这里使用的连接是 IPC ，因为调用其他进程是在同一个电脑，所以这时使用 IPC 的效率会比 http 和 tcp 高。原因是 IPC 是进程间通信，效率和内存共享差不多。而使用 http 或 tcp 需要把信息发送给本地巡回，然后再返回。而且使用 http 需要做额外的过程，需要走 http 的协议。使用 tcp 需要使用握手，性能都比 IPC 差。

## 运行的类

所有需要在 C++ 程序运行的类都需要注册，因为C++程序需要找到程序集所有符合的类，所以需要这些类标记

```csharp
    /// <summary>
    /// 放在远程的实例
    /// <remarks>请不要在代码保存实例</remarks>
    /// </summary>
    interface IRemote
    {

    }
```

```csharp
    /// <summary>
    ///  共享使用的类，这个类会在远程进程创建
    /// </summary>
    [AttributeUsage(AttributeTargets.Class)]
    public class RemoteAttribute : Attribute
    {

    }
```

例如有一个类需要在 C++ 程序运行，在 WPF 程序使用，那么就需要这样写

```csharp
    [Remote]
    public class BaltrartularLouronay : MarshalByRefObject
    {
        public void TerecaLesi()
        {
            Console.WriteLine("调用");
        }
    }
```

继承 MarshalByRefObject 标记 Remote 就可以了

## 运行C++程序

运行需要获得程序所有类，需要在C++程序使用的类，实现它。

```csharp
    /// <summary>
    /// 远程本机进程
    /// </summary>
    public class RemoteNative
    {
        /// <summary>
        ///  加载程序集
        /// </summary>
        /// <param name="args">传入参数</param>
        /// <param name="assembly"></param>
        public void Run(string[] args, Assembly assembly)
        {
            Parser.Default.ParseArguments<Options>(args).MapResult(options =>
            {
                Console.WriteLine("端口号" + options.Port);
                new Thread(() =>
                {
                    Console.WriteLine("启动库");
                    Run(options.Port, assembly);
                    Console.WriteLine("启动完成");
                }).Start();
                return 0;
            }, _ => -1);
        }

        /// <summary>
        ///  加载程序集
        /// </summary>
        /// <param name="port">端口</param>
        /// <param name="assembly">程序集</param>
        public void Run(string port, Assembly assembly)
        {
            _channel = Terminal.CreatChannel(port);

            ChannelServices.RegisterChannel(_channel, false);

            //设置租用管理器的初始租用时间为无限 http://www.cnblogs.com/wayfarer/archive/2004/08/05/30437.html
            LifetimeServices.LeaseTime = TimeSpan.Zero;

            //注册实例
            var remoteProcessBrake = new RemoteProcessBrake();

            remoteProcessBrake.Exit += RemoteProcessBrake_Exit;

            // 防止对象回收
            // 如果不使用 var objRef = x 那么在运行就发现 System.Runtime.Remoting.RemotingException:“找不到请求的服务”
            var objRef = RemotingServices.Marshal(remoteProcessBrake, remoteProcessBrake.GetType().Name);

            _objRefList.Add(objRef);

            Init(assembly);
        }

        private void RemoteProcessBrake_Exit(object sender, EventArgs e)
        {
            Environment.Exit(0);
        }

        private void Init(Assembly assembly)
        {
            foreach (var temp in assembly.GetTypes().Where(temp => temp.GetCustomAttribute<RemoteAttribute>() != null))
            {
                var obj = CreateInstance(temp);
                if (obj != null)
                {
                    // 防止对象回收
                    var objRef = RemotingServices.Marshal(obj, temp.Name);
                    _objRefList.Add(objRef);
                }
            }
        }

        private MarshalByRefObject CreateInstance(Type type)
        {
            ConstructorInfo constructor = type.GetConstructor(Type.EmptyTypes);
            return constructor == null ? null : Activator.CreateInstance(type) as MarshalByRefObject;
        }

        /// <summary>
        /// 防止对象回收
        /// </summary>
        private List<ObjRef> _objRefList = new List<ObjRef>();

        private IChannel _channel;
    }

```

## 通道

如果需要两个程序连接，需要创建通道

```csharp
   /// <summary>
    /// 通道
    /// </summary>
    public class Terminal
    {
        /// <summary>
        /// 创建连接
        /// </summary>
        /// <param name="port">对于服务端需要一个标示符，对于客户端请使用空</param>
        /// <returns></returns>
        public static IChannel CreatChannel(string port = "")
        {
            if (string.IsNullOrEmpty(port))
            {
                port = Guid.NewGuid().ToString("N");
            }

            var serverProvider = new BinaryServerFormatterSinkProvider();
            var clientProvider = new BinaryClientFormatterSinkProvider();
            serverProvider.TypeFilterLevel = TypeFilterLevel.Full;
            IDictionary props = new Hashtable();
            props["portName"] = port;

            return new IpcChannel(props, clientProvider, serverProvider);
        }
    }
```

对于 WPF 程序只需要创建随机的端口，对于 C++ 程序需要创建 WPF 程序告诉他的端口，这样 WPF 程序才可以发送数据到 C++ 程序

## 使用

尝试把上面的类复制到自己的一个项目，然后创建两个项目，一个是 WPF 程序，一个是C++程序，让两个程序都引用这个项目。

注意创建的项目需要引用 System.Runtime.Remoting 

例如创建 MairzearPowhel 程序做 WPF 程序用来调用 SedreaSudome 程序。在 MairzearPowhel 需要引用 SedreaSudome 可以获得里面的类而且用来启动 SedreaSudome 。

```csharp
    /// <summary>
    /// MainWindow.xaml 的交互逻辑
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        private void SouhaiNosoja()
        {
            // 启动C++程序
            var remoteProcessManager = new RemoteProcessManager("SedreaSudome.exe");

            // 连接
            remoteProcessManager.Connect();

            var baltrartularLouronay = remoteProcessManager.GetObject<BaltrartularLouronay>();

            // 执行里面方法
            baltrartularLouronay.TerecaLesi();
        }

        private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            SouhaiNosoja();

        }
    }

```

对于 C++ 程序只需要几个代码

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            Debugger.Launch();

            var remoteNative = new RemoteNative();
            remoteNative.Run(args, typeof(Program).Assembly);

            while (true)
            {
                Console.Read();
            }
        }
    }
```

如果发现无法使用，请联系我

![](https://i.loli.net/2018/06/24/5b2f3c70dd5e9.jpg)

感谢 [洪三水](https://www.pixiv.net/member_illust.php?mode=medium&illust_id=69353928)提供图片

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
