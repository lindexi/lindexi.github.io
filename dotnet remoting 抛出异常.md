# dotnet remoting 抛出异常

本文告诉大家如何在使用 .net remoting 的时候，抛出异常。

<!--more-->
<!-- CreateTime:2019/9/24 15:39:50 -->

<!-- 标签：.net remoting,rpc,wpf -->

<div id="toc"></div>

所有在远程软件运行的类，如果需要传输到本地，都需要继承 MarshalByRefObject 或其他可以序列化的类。

在 .net Framework 4.0 就默认指定只反序列化基础类型，如果需要反序列化其他的类型，那么就需要设置`TypeFilterLevel`，设置的方法是在使用下面代码

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

但是设置了TypeFilterLevel不是对所有的类型都可以进行转换，如果不小心直接在调用方法抛出异常，那么会因为无法反序列，让本地拿不到

```csharp
 // 远程

 public void Foo()
 {
 	throw new CsdnNotFoundException();
 }

 public class CsdnNotFoundException : Exception
 {
 	public CsdnNotFoundException(string str) :
 	       base(str)
 	{

 	}       
 }
```

这时本地会提示`System.Runtime.Serialization.SerializationException`程序无法序列。

如果需要在 .net remoting 使用异常，那么需要自己创建一个异常，继承 RemotingException

## 反序列

因为默认的 RemotingException 没有反序列，所以需要添加 Serializable 特性

```csharp
 [Serializable]
 public class CsdnNotFoundException : RemotingException
 {
 	public CsdnNotFoundException(string str) :
 	       base(str)
 	{

 	}       
 }
```

微软建议继承`ISerializable`，标记特性

```csharp
 [Serializable]
 public class CsdnNotFoundException : RemotingException, ISerializable
 {
 	public CsdnNotFoundException(string str) :
 	       base(str)
 	{

 	}       
 }
```

如果直接运行，会发现报告`System.Runtime.Serialization.SerializationException:“未找到反序列化“lindexi.Csdn.CsdnNotFoundException”类型对象的构造函数`

解决方法是创建一个构造函数，写入这个函数就不需要再写其他的代码。

```csharp
        protected CsdnNotFoundException([NotNull] SerializationInfo info, StreamingContext context) : base(info,
            context)
        {
        }
```

如果有一些特殊的属性需要自己设置，建议创建一个默认构造函数，和两个方法，因为使用上面的方法不会序列化自己定义的属性。

```csharp
 [Serializable]
 public class CsdnNotFoundException : RemotingException, ISerializable
 {
    public CsdnNotFoundException()
    {
    	//默认构造，可以在反射创建
    }

 	public CsdnNotFoundException(string str) :
 	       base(str)
 	{

 	}      

 	      protected CsdnNotFoundException([NotNull] SerializationInfo info, StreamingContext context) 
 	      //: base(info, context) 不使用基类的原因是基类会报告 找不到 ClassName 和其他很多的坑
        {
            //反序列化创建

            Message = (string) info.GetValue(MessageSerialization, typeof(string));
        } 

        // 重写消息，用于在构造设置值
        public override string Message { get; }

        // 用于在构造拿到消息的值
        private const string MessageSerialization = "Message";

        // 重写这个方法，在序列化调用
        public override void GetObjectData(SerializationInfo info, StreamingContext context)
        {
            info.AddValue(MessageSerialization, Message);
        }
 }
```

在 GetObjectData 拿到必要的属性，这个需要自己把需要的属性写入。然后在构造函数重写`[NotNull] SerializationInfo info, StreamingContext context`方法的，可以拿到值

因为上面的代码用到 Message ，需要重写这个属性，因为默认是只读，不能在构造函数设置。

是不是觉得很复杂，实际上简单的方法是通过 json 在GetObjectData把类转换为json，在构造转换为类。

## ISerializable

那么为什么在使用 Serializable 特性还需要继承 ISerializable ，因为继承 ISerializable 就可以在一个构造函数`xx([NotNull] SerializationInfo info, StreamingContext context)`进行处理和处理如何序列化。处理如何序列化可以提高性能，因为自己知道哪些需要序列化，哪些不需要。

关于 ISerializable 请看 [c# - What is the point of the ISerializable interface? - Stack Overflow](https://stackoverflow.com/questions/810974/what-is-the-point-of-the-iserializable-interface )

[How to: Create an Exception Type That Can be Thrown by Remote Objects](https://msdn.microsoft.com/en-us/library/s9fyb186(v=vs.100).aspx )

![](https://i.loli.net/2018/04/08/5ac9ffb44217b.jpg)

我的博客即将搬运同步至腾讯云+社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite_code=19bm8i8js1ezb

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
