
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




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。