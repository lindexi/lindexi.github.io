
# dotnet 集成测试 SmartSql 存在静态量导致多个主机启动提示 Alias 已存在

在集成测试中，我采用单个进程开启多个主机，可以理解为一个用例开启一次主机。但是在我使用到 SmartSql 的功能时，我发现在一个主机关闭之后，再开启下一个主机，会因为使用了相同的别名而提示 SmartSql.Alias:[SmartSql] already exist 错误。本文告诉大家如何解决此问题

<!--more-->


<!-- 发布 -->

默认的 dotnet 应用在主机销毁的时候，都会销毁自己的资源。而 SmartSql 没有做这个处理，同时使用了静态量，这就意味着两个不同的主机都会访问到相同的对象，在开启第二个主机的时候，会因为存在相同的 Alias 而提示出错，如下面代码

```
λ:SmartSql.ISqlMapper -> λ:SmartSql.SmartSqlBuilder.
                      ---> SmartSql.Exceptions.SmartSqlException: SmartSql.Alias:[SmartSql] already exist.
                        at SmartSql.SmartSqlContainer.Register(SmartSqlBuilder smartSqlBuilder)
                        at SmartSql.SmartSqlBuilder.Build()
                        at Microsoft.Extensions.DependencyInjection.SmartSqlDIExtensions.<>c__DisplayClass0_0.<AddSmartSql>b__0(IServiceProvider sp)
                        at Autofac.Extensions.DependencyInjection.AutofacRegistration.<>c__DisplayClass3_0.<Register>b__0(IComponentContext context, IEnumerable`1 parameters)
```

因为在应用中使用静态量的逻辑不少，因此集成测试可以采用多进程方法，一个进程跑一个主机，在一个主机里面跑多个用例。这样的优势可以减少静态清理，缺点是集成测试项目要么有很多个，要么需要一个中间的管理，相对复杂

第二个方法是给 SmartSql 一个别名，如下面代码

```csharp
      services.AddSmartSql((sp, builder) =>
      {
            builder.UseAlias("随机的命名");
      });
```

保持每次的主机都使用不同的 随机的命名 就可以解决此问题

特别感谢 [小黑兔](173592829) 的帮助

欢迎加入 SmartSql 官方群 604762592

关于集成测试请看 [asp dotnet core 基于 TestServer 做集成测试](https://blog.lindexi.com/post/asp-dotnet-core-%E5%9F%BA%E4%BA%8E-TestServer-%E5%81%9A%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。