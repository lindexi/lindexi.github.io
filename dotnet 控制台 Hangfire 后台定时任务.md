# dotnet 控制台 Hangfire 后台定时任务

本文告诉大家如何在 dotnet core 的控制台通过 Hangfire 开启后台定时任务

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


首先需要安装 [HangFire](https://www.nuget.org/packages/Hangfire ) 这个 Nuget 库，通过这个库可以用来做定时任务，虽然很多时候都是在 ASP.NET Core 后台

默认后台任务需要数据库，用的数据库是 SqlServer 但是我觉得没有多少小伙伴想在自己的控制台项目直接使用 SQLServer 所以需要再安装 Hangfire.SQLite 通过本地文件 SQLite 做数据库

在开始使用之前，需要配置使用的数据库文件，通过下面的代码就可以使用当前工作文件夹的 `CalelsairstirKislezootaima.db` 作为 SQLite 数据文件

```csharp
            GlobalConfiguration.Configuration.UseSQLiteStorage("Data Source=./CalelsairstirKislezootaima.db;");

```

需要注意，在 `UseSQLiteStorage` 是可以传入配置的 name 和数据文件连接字符串，如果字符串的最后不是使用 `;` 结束，那么将会被认为是一个 name 将会从配置里面尝试读取

在控制台开始后台任务需要先开启 BackgroundJobServer 这个类是可以被释放的，可以尝试这样写

```csharp
            using (new BackgroundJobServer())
            {

            }
```

开始一个不断执行的任务

```csharp
            using (new BackgroundJobServer())
            {
                var jobId = BackgroundJob.Enqueue(
                    () => Console.WriteLine("Fire-and-forget!"));

                Console.Read();
            }
```

为什么这里需要 `Console.Read` 因为如果退出了 `BackgroundJobServer` 那么后台任务就不在运行

开始一个一秒运行一次的任务

```csharp
            using (new BackgroundJobServer())
            {
                BackgroundJob.Schedule(() => Console.WriteLine("Reliable!"), TimeSpan.FromSeconds(1));
                Console.Read();
            }
```

代码 [https://github.com/lindexi/lindexi_gd/tree/e397171412e1cae803b8590ffd31413027866dc9/SeaherehorjawKitirnaivouwebooca](https://github.com/lindexi/lindexi_gd/tree/e397171412e1cae803b8590ffd31413027866dc9/SeaherehorjawKitirnaivouwebooca)

更多方法请看 [Hangfire.SQLite An easy way to perform fire-and-forget, delayed and recurring tasks inside ASP.NET apps](https://github.com/tidusjar/Hangfire.SQLite )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
