# C＃ 写系统日志

因为我不想使用自己写文件，我的软件是绿色的，所以把日志写到 Windows 日志。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->

<!-- 标签：wpf，WPF调试 -->

首先告诉大家什么是系统日志，请看下面，这就是我要告诉大家的日志。写在这里就把日志放在 系统日志那里，看起来很厉害。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20181139593.jpg)

如果需要写日志，需要管理员权限，如果没有权限会出现下面异常

```csharp
System.Security.SecurityException:“未找到源，但未能搜索某些或全部事件日志。 不可访问的日志: Security
```

需要判断当前是否已经存在日志，下面我来创建一个事件叫 “德熙” 

```csharp
            if (EventLog.SourceExists("德熙"))
            {
                EventLog.CreateEventSource("德熙", "Application");
            }
```

这里的 Application 就是写到哪个，一般都是选 Application ，可以从图片看到系统的有应用程序、安全、Setup、系统几个日志，程序一般都是写到程序

## 写日志

如果已经创建了日志事件，那么继续来写入日志就不需要管理员权限了。所以在安装程序的过程创建日志就可以让程序不需要再写入日志时需要管理员权限。

写入可以使用 WriteEntry ，需要传入写入的日志和内容

```csharp
            EventLog.WriteEntry("德熙", "有个不愿告诉你名称的程序在这里写字符串");

```

这个方法还有几个重载，可以传入日志类型，是成功、失败还是其他。还可以传入 id ，通过id 可以找到为什么需要写日志，不过需要在自己定义，还可以添加附件，于是我就不需要自己写文件日志。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201811310950.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。