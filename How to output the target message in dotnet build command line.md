# How to output the target message in dotnet build command line

How can I output my target message when I using dotnet build in command line.

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


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