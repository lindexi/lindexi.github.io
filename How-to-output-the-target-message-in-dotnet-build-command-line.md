
# How to output the target message in dotnet build command line

How can I output my target message when I using dotnet build in command line.

<!--more-->



I use command line to create a web api application.

```csharp
dotnet new webapi -o Lindexi
```

Then I edit the `Lindexi\Lindexi.csproj` and add the message.

```csharp
  <Target Name="Lindexi" BeforeTargets="CoreCompile">
    <Message Text="Welcome to my blog" />
  </Target>
```

I use the `dotnet build` to build the application but I can not find the message.

I try to change the message to warning that I can find the text in output.

In dotnet command, the `verbosity` can be set to verbosity level of the command.

The Allowed values are `q[uiet]`, `m[inimal]`, `n[ormal]`, `d[etailed]`, and `diag[nostic]`.

The default value is minimal that do not print the target message. The lowest level of output message is normal command.

```csharp
dotnet build -v n
```

[dotnet build command - .NET Core CLI](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-build?tabs=netcore2x )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。