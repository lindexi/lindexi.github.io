
# .net remoting 使用事件

在RPC如果需要使用事件，相对是比较难的。本文告诉大家如何在 .net remoting 使用事件。

<!--more-->


<!-- csdn -->
<!-- 标签：.net remoting,rpc,wpf -->

<div id="toc"></div>

在我这个博客[WPF 使用RPC调用其他进程](./WPF-%E4%BD%BF%E7%94%A8RPC%E8%B0%83%E7%94%A8%E5%85%B6%E4%BB%96%E8%BF%9B%E7%A8%8B.html )已经有告诉大家如何简单使用。

但是对于事件的使用还是没有详细告诉大家。

先来写一个简单的代码，需要创建三个项目，一个存放的是其他进程，一个是库，另一个是呆磨。

在上个文章告诉大家的时候没有告诉大家使用的 Channel 的方式，下面让我来告诉大家如何使用 Channel

## 使用 Channel

实际上可以使用的 Channel 是有很多，可以自己定义，但是建议使用的有三个

- HttpChannel 功能比较强大，支持在广域网使用，可以让很多不是 .net 写的程序使用，但是需要自己写安全的代码

- TcpChannel 速度更快的方式，一般在局域网使用

- IpcChannel 就在相同的机器内使用，速度最快，使用的是微软系统系统的方法

所有的 Channel 都需要传入 port ，但是不是所有的类型都是 int ，其中 HttpChannel 和 TcpChannel使用的都是 int ，一般给的空闲的端口。而 IpcChannel 需要的是一个字符串，可以给他一个随机的字符串。

## 序列化

如果简单写一个类，使用了这个类里的事件，那么一般会出现异常

```csharp
程序集“林德熙.RemoteProcess.Demo, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null”中的类型“林德熙.RemoteProcess.Demo.MainWindow”未标记为可序列化
```

为了可以使用事件，需要先修改 Channel ，下面我使用的是 IpcChannel 

写一个方法来创建连接，写在库项目，这个方法在呆磨和其他进程需要使用，原来创建相同的方法进行连接

```csharp
        public static IChannel CreatChannel(string port = "")
        {
            if (string.IsNullOrEmpty(port))
            {
                port = Guid.NewGuid().ToString("N");
            }

            var serverProvider = new SoapServerFormatterSinkProvider();
            var clientProvider = new SoapClientFormatterSinkProvider();
            serverProvider.TypeFilterLevel = TypeFilterLevel.Full;
            IDictionary props = new Hashtable();
            props["portName"] = port.ToString();

            return new IpcChannel(props, clientProvider, serverProvider);
        }
```

代码需要使用 TypeFilterLevel 设置，默认使用的是Low，所以会出现事件无法序列化。

其实传入的 serverProvider等 可以使用 BinaryServerFormatterSinkProvider 类型，一般推荐使用 SoapServerFormatterSinkProvider ，他的速度比较快。

这时呆磨使用的创建就不需要写端口

```csharp
            _channel = Terminal.CreatChannel();//客户端

            ChannelServices.RegisterChannel(_channel, false);
```

其他进程需要指定一个端口，这时呆磨传入的，因为呆磨需要知道其他进程使用的才可以

```csharp
       _channel = Terminal.CreatChannel(port);

            ChannelServices.RegisterChannel(_channel, false);
```

一般在 IpcChannel 都是说连接是不安全的，因为有很多特殊的软件都会发送一些信息让软件通信失败

因为序列化需要知道类的属性，所以需要在获得事件，重新使用一个类来获得

需要在库定一个两个类，一个是 Foo ，也就是需要获得事件的类，另一个是 F1 用于给呆磨转消息

```csharp
    //库
    public class Foo : MarshalByRefObject
    {
        public event EventHandler F1;
    }
```

```csharp
  //其他进程

              _channel = Terminal.CreatChannel(port);

            ChannelServices.RegisterChannel(_channel, false);

            var obj = new Foo();
                      ObjRef objRef = RemotingServices.Marshal(obj, temp.Name);

```

```csharp
 //呆磨
        public void Connect()
        {
            //启动远程进程
            ProcessId = Process.Start("林德熙.RemoteProcess.exe", "-p " + Port)?.Id ?? -1;

            _channel = Terminal.CreatChannel();//客户端

            ChannelServices.RegisterChannel(_channel, false);
        }

        public T GetObject<T>()
        {
            CheckProcess();
            return (T) Activator.GetObject(typeof(T),
                   "Ipc://" + Port + "/" + typeof(T).Name);
        }

                    GetObject<Foo>().F1 += MainWindow_F1; //出现异常

```

因为没有把呆磨序列，只能再新建一个类 F1

```csharp
  // 库
     public delegate void F2(object obj, string str);

    [Remote]
    public class Foo : MarshalByRefObject
    {
        public event F2 F1;

        public virtual void OnF1()
        {
            F1?.Invoke(this, "cnblogs");
        }
    }

    public class F1 : MarshalByRefObject
    {
        public event EventHandler<string> Foo;

        public void OnF1(object sender, string e)
        {
            Foo?.Invoke(sender, e);
        }
    }
```

运行的时候，两个类所在的是 Foo 在其他进程，而 F1 在呆磨程序

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018126153031.jpg)

使用的时候需要这样写

```csharp
            var f = GetObject<Foo>();
            F1 f1 = new F1(); //创建一个类来直接获得事件，不能直接添加呆磨程序中的函数，必须创建另一个类
            f.F1 += f1.OnF1; 
            f1.Foo += Foo; //这个类的事件给呆磨

           private void Foo(object sender, string s2)
        {

        }
```

可以看到运行`f.OnF1();`就可以让呆磨Foo获得值

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018126154854.jpg)

从上面代码看到，为什么不使用 `EventHandler<string>` ，自己定义委托，一般都是不建议自己定义，但是这里需要自己定义的，因为如果使用 `EventHandler<string>`会出现异常

Soap 序列化程序不支持序列化一般类型: System.EventHandler\`1[System.String]。

这就是用事件的方法，需要记得

在库创建两个类，一个类用于从其他进程发送事件给呆磨，另一个类用于接收这个事件，把事件转发给呆磨

原因是在使用 `+=` 需要序列化右边的这个类，而如何直接对 Foo 类进行添加事件，那么需要序列化呆磨。然而呆磨没有放在库，而且其他进程没有引用呆磨，所以其他进程无法序列呆磨的类型。但是在库写另一个类F1，其他进程可以序列化F1，所以可以获得在呆磨创建的F1。把事件给在呆磨创建的F1，让F1转发事件给呆磨。

实际上使用的时候就比直接使用需要加一个新的类，而且不能直接使用`EventHandler<string>` 

为什么不能使用 `EventHandler<string>`  原因是 SoapServerFormatterSinkProvider 不支持泛型，可以使用 BinaryServerFormatterSinkProvider 的方法
	



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。