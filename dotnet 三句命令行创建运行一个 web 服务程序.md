# dotnet 三句命令行创建运行一个 web 服务程序

现在 dotnet 的服务创建十分具有效率，本文的前提要求是电脑上面已经安装了 dotnet 程序，接下来就是三句命令行的事情

<!--more-->
<!-- CreateTime:2020/1/13 8:45:47 -->

<!-- 发布 -->

如果还没有安装 dotnet 那么请到 [https://dotnet.microsoft.com/ 官网](https://dotnet.microsoft.com/ ) 下载安装，基本上看界面就知道如何下载安装

接下来在可以进行测试的临时文件夹打开命令行，这一句话不算在本文的命令行数量统计内

第一句话创建一个 web 服务程序的代码到 Foo 文件夹

```
dotnet new webapi -o Foo
```

这里的 new 就是创建的意思，而 webapi 指的是创建的是什么样的模板的代码，后续加上的 `-o` 表示创建到哪个文件夹，这里指定创建到 Foo 文件夹里面

第二句话就是进入 Foo 文件夹

```
cd Foo
```

第三句话就是运行刚才创建的代码，第一次运行编译 dotnet 项目需要等待一下依赖包的下载

```
dotnet run
```

此时就完成了一个简单的服务的创建和运行了，如果看到下面代码表示服务已经运行起来，可以访问

```
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: http://localhost:5000
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: f:\lindexi\foo\

```

现在创建的只是一个web api的服务，最简单的访问方法是通过浏览器访问，通过浏览器访问 `http://localhost:5000/WeatherForecast` 就可以看到返回值了

现在只需要使用三句命令行就完成了一个 asp dotnet core 服务的创建和运行，实在简单

也许此时有问题是，我访问的这个链接是如何对应到代码里面的？请打开 Foo 文件夹里面的 Controllers 文件夹，此时可以看到 默认的 `xxController.cs` 文件，在 dotnet core 3.1 的版本默认写的是 `WeatherForecastController.cs` 文件

此时不需要使用 VisualStudio 可以使用 VisualStudio Code 或 SublimeText 或 NotePad++ 打开这个 WeatherForecastController.cs 文件，可以看到这里有一个 Get 方法，其实刚才在浏览器看到的内容就是通过这个 Get 方法返回的。尝试将这个 Get 方法修改一下

```csharp
        [HttpGet]
        public string Get()
        {
            return "林德熙是逗比";
        }
```


此时关闭原有的控制台，重新打开控制台，进入 csproj 所在的文件夹，执行 `dotnet run` 命令重新运行。当然不想关闭原有的控制台，也可以按下 `ctrl+c` 键，也就是复制的快捷键，结束当前运行的程序，结束之后重新输入 dotnet run 重新运行就可以

刷新一下网页，访问 `http://localhost:5000/WeatherForecast` 就可以看到 林德熙是逗比 的字符串

是不是超级简单

接下来可能会问的问题就是，我如何修改浏览器对应的链接，让新的链接可以对应到新的方法？我如何修改默认的端口，我想要开放的是 13802 端口可以怎么做？我想要让小伙伴也能访问我的服务，我可以怎么做？

这些问题都十分简单，微软提供了超级体系的学习平台，此平台是免费的，不仅有文档知识同时提供免费虚拟机可以实践。点击[这个链接访问官方教程](https://docs.microsoft.com/zh-cn/learn/modules/build-web-api-net-core/ ) 我在学习平台升级 6 级了，欢迎小伙伴组队

更多请看：

- [如何设置 ASP.NET Core 程序监听的 IP 和端口 - walterlv](https://blog.walterlv.com/post/configure-urls-and-port-for-asp-dotnet.html )
- [为 ASP.NET Core 程序制作 URL 的 301/302 跳转 - walterlv](https://blog.walterlv.com/post/redirect-middleware-for-asp-dotnet.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
