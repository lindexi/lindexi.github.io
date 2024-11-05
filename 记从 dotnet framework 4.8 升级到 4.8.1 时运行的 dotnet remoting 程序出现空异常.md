# 记从 dotnet framework 4.8 升级到 4.8.1 时运行的 dotnet remoting 程序出现空异常

本文记录一个奇怪的坑，某台用户设备从 .NET Framework 4.8 更新到 .NET Framework 4.8.1 时，所运行的 .NET Remoting 程序出现了奇怪的空异常。且重启之后不复现

<!--more-->
<!-- CreateTime:2024/11/05 07:11:44 -->

<!-- 发布 -->
<!-- 博客 -->

错误堆栈如下

```
System.NullReferenceException:“Object reference not set to an instance of an object.”
   在 System.Runtime.Remoting.Proxies.RealProxy.HandleReturnMessage(IMessage reqMsg, IMessage retMsg)
   在 System.Runtime.Remoting.Proxies.RealProxy.PrivateInvoke(MessageData& msgData, Int32 type)
```

```
System.NullReferenceException:“Object reference not set to an instance of an object.”
   在 System.Runtime.Remoting.Messaging.StackBuilderSink._PrivateProcessMessage(IntPtr md, Object[] args, Object server, Object[]& outArgs)
   在 System.Runtime.Remoting.Messaging.StackBuilderSink.SyncProcessMessage(IMessage msg)
```

具体原因未知，预计是版本之间的兼容性问题，导致某些逻辑/进程跑了不同的 .NET Framework 版本。而多个 .NET Framework 版本没有做好运行时兼容性