# HttpRepl 互操作的 RESTful HTTP 服务调试命令行工具

今天早上曽根セイラ告诉我一个好用的工具 HttpRepl 这是一个可以在命令行里面对 RESTful 的 HTTP 服务进行路由跳转和访问的命令行工具。可以使用 cd 这个命令和像文件跳转已经跳转到下一级的路由，也可以通过 dir 命令找到同一级的路由，同时支持集成到 VisualStudio 和 VisualStudio Code 里面

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


最近 ASP.NET 团队创建了一个叫 [HttpRepl](https://github.com/aspnet/HttpRepl) 的命令行工具，通过这个命令行工具可以像访问文件或文件夹一样访问 HTTP 服务。通过在命令行给一个入口的地址，然后就可以执行 `dir` 和 `cd` 两个命令分别用于枚举和跳转的功能

使用方法是先通过 `dotnet tool install` 安装这个工具，安装代码如下

```csharp
dotnet tool install -g Microsoft.dotnet-httprepl --version “3.0.0-*”
```

安装完成之后，则可以使用如下命令指定入口

```csharp
dotnet httprepl 入口URL

```

如官方的例子知道了访问 `http://localhost:65369/` 链接

```csharp
dotnet httprepl http://localhost:65369/
```

如果在执行以上命令的时候提示没有安装足够的 .NET Core 框架版本，那么请自行从[官方](https://dotnet.microsoft.com/download) 下载最新版本和最新预览版本安装

```csharp
It was not possible to find any compatible framework version
The specified framework 'Microsoft.NETCore.App', version '3.0.0-preview6-27804-01' was not found
```

设置之后则可以通过 `dir` 查看同一级的路由里面有哪些可访问的链接

通过输入 `get` 命令就可以直接拉取，用起来十分简单，详细使用方法请下载安装之后输入以下命令就可以看到

```csharp
dotnet httprepl http://blog.lindexi.com
help
```

看到的控制台输出大概如下

```csharp
C:\Users\lindexi>dotnet httprepl http://blog.lindexi.com
(Disconnected)~ set base "http://blog.lindexi.com"

http://blog.lindexi.com/~ help

HTTP Commands:
Use these commands to execute requests against your application.

GET            get - Issues a GET request
POST           post - Issues a POST request
PUT            put - Issues a PUT request
DELETE         delete - Issues a DELETE request
PATCH          patch - Issues a PATCH request
HEAD           head - Issues a HEAD request
OPTIONS        options - Issues a OPTIONS request

set header     Sets or clears a header for all requests. e.g. `set header content-type application/json`

Navigation Commands:
The REPL allows you to navigate your URL space and focus on specific APIs that you are working on.

set base       Set the base URI. e.g. `set base http://locahost:5000`
set swagger    Sets the swagger document to use for information about the current server
ls             Show all endpoints for the current path
cd             Append the given directory to the currently selected path, or move up a path when using `cd ..`

Shell Commands:
Use these commands to interact with the REPL shell.

clear          Removes all text from the shell
echo [on/off]  Turns request echoing on or off, show the request that was made when using request commands
exit           Exit the shell

REPL Customization Commands:
Use these commands to customize the REPL behavior.

pref [get/set] Allows viewing or changing preferences, e.g. 'pref set editor.command.default 'C:\\Program Files\\Microsoft VS Code\\Code.exe'`
run            Runs the script at the given path. A script is a set of commands that can be typed with one command per line
ui             Displays the Swagger UI page, if available, in the default browser
```

如何在 VisualStudio 集成，在每次调试的时候访问的不是浏览器而是命令行就请看 ZaraNet 大佬的博客

[使用Http-Repl工具测试ASP.NET Core 2.2中的Web Api项目 - ZaraNet - 博客园](https://www.cnblogs.com/ZaraNet/p/10448247.html )


[HttpRepl: A command-line tool for interacting with RESTful HTTP services](https://devblogs.microsoft.com/aspnet/httprepl-a-command-line-tool-for-interacting-with-restful-http-services/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
