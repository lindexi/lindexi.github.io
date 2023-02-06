# dotnet 6 命令行 cmd 设置输出英文解决中文乱码

我遇到在部署 CI 服务器，执行 cmd 命令构建，输出的中文是乱码。我期望让 dotnet 命令行输出使用英文解决乱码问题。通过设置 dotnet 命令行的语言文化，即可解决此问题

<!--more-->
<!-- CreateTime:2023/2/2 16:31:36 -->

<!-- 发布 -->
<!-- 博客 -->

给 dotnet.exe 进程设置以下环境变量即可

```csharp
DOTNET_CLI_UI_LANGUAGE=en-US
```

如在 CMD 下，可以使用以下代码设置环境变量，如此设置的环境变量只影响当前的 cmd 控制台和在此控制台设置之后启动的应用

```csharp
set DOTNET_CLI_UI_LANGUAGE=en-US
dotnet --info
```

可以看到 `dotnet --info` 输出的就是英文

反过来，如果期望让 dotnet 命令行输出中文，可以设置语言文化为中文

```csharp
set DOTNET_CLI_UI_LANGUAGE=zh-CN
```

参考

[Use UTF-8 Encoding by Default On Windows On Non English Languages by nagilson · Pull Request #29755 · dotnet/sdk](https://github.com/dotnet/sdk/pull/29755 )

[dotnet cli produces `?`s in output · Issue #8833 · dotnet/sdk](https://github.com/dotnet/sdk/issues/8833 )
