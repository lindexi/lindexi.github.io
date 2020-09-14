
# dotnet 如何调试 SmartSql 的实际执行 SQL 语句

在使用 SmartSql 中的 SQL 语句是支持进行属性替换，在调试时如何拿到实际执行的 SQL 命令

<!--more-->


<!-- CreateTime:2020/9/12 12:06:22 -->

<!-- 发布 -->

只需要在 appsettings.json 中将 LogLevel 的 Default 设置为 Debug 等级，可以在运行时看到从开发者编写的 SQL 语句加上参数的实际 SQL 语句，大概内容如下

```csharp
dbug: SmartSql.Middlewares.PrepareStatementMiddleware[0]
      Statement.Id:[User.GetEntity],Sql:
      Select * From T_User Where Id=@Id
      Parameters:[Id=1]
      Sql with parameter value:
      Select * From T_User Where Id=1
```


如果存在 appsettings.Development.json 文件，那么请在调试时更改 appsettings.Development.json 文件，大概代码如下

```json
{
  "Logging": 
  {
    "LogLevel":
     {
      "Default": "Debug",
      "System": "Information",
      "Microsoft": "Information"
    }
  }
}
```

本文使用的 SmartSql 是在 GitHub 完全开源 [https://github.com/dotnetcore/SmartSql](https://github.com/dotnetcore/SmartSql)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。