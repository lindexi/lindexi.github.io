
# dotnet 关于 SmartSql 的 SQL 语句的属性替换前缀说明

本文将告诉大家在 SmartSql 的 SQL 语句的属性前缀 ParameterPrefix 的默认值和用法以及原理

<!--more-->


<!-- CreateTime:2020/9/12 11:16:01 -->



## 用途

使用 SmartSql 库的属性前缀 ParameterPrefix 能赋予 SQL 语句属性替换参数的功能，可以将 SQL 语句中的属性替换为业务方传入的参数

如以下代码，在运行时将替换 `@Id` 为业务传入参数

```xml
        <!--获取表映射实体-->
        <Statement Id="GetEntity">
            Select * From T_User Where Id=@Id
        </Statement>
```

## 用法

在 SmartSql 中将使用两套属性前缀 ParameterPrefix 定义，一套是根据具体的数据库采用不同的默认属性前缀 ParameterPrefix 字符。另一套是开发者自定义的属性前缀，开发者可以在 SmartSqlMapConfig.xml 的 Settings 的 ParameterPrefix 属性进行自定义，如以下示例，将定义默认的属性前缀为 `$` 符号

```xml
<SmartSqlMapConfig xmlns="http://SmartSql.net/schemas/SmartSqlMapConfig.xsd">
    <Settings ParameterPrefix="$"/>
</SmartSqlMapConfig>
```

## 默认值

默认的各个数据库采用的默认属性前缀 ParameterPrefix 字符如下

- SQL Server: `@`
- My SQL Server: `@`
- POSTGRESQL: `@`
- SQLite: `@`
- MySQL: `?`
- MySQL CONNECTOR: `?`
- ORACLE: `:`

## 原理

在 SmartSql 库的一个重要功能就是支持编写底层的 SQL 语句。在 SQL 语句里面的属性可以通过一定的规则替换为业务层传入的参数。如下面代码

```xml
        <!--获取表映射实体-->
        <Statement Id="GetEntity">
            Select * From T_User Where Id=@Id
        </Statement>
```

以上代码的 `Where Id=@Id` 的 `@Id` 将会在运行时替换为映射参数的具体值。映射参数在对应的 Repository interfaces 定义，如以下代码示例

```csharp
    public interface IUserRepository
    {

        [Statement(Id = "GetEntity")]
        User GetById([Param("Id")]long id);

    }
```

以上代码示例可以在[官方仓库的 sample/SmartSql.Sample.AspNetCore/DyRepositories/IUserRepository.cs](https://github.com/dotnetcore/SmartSql/blob/e3a18b7c356b7a2033eaf3c9a55f9ca9b92121ec/sample/SmartSql.Sample.AspNetCore/DyRepositories/IUserRepository.cs#L16-L17) 找到

那么 SmartSql 是如何了解需要将 `Where Id=@Id` 的 `@Id` 替换为 IUserRepository 的 `User GetById([Param("Id")]long id);` 方法中的 `long id` 参数？请看下文

可以关注到在 `Select * From T_User Where Id=@Id` 中需要替换的属性包含了前缀 ParameterPrefix 属性前缀 `@` 符号。在 SmartSql 底层将会使用正则对此字符串进行替换，能够通过前缀取出需要替换的属性。而根据方法里面的 `Param` 特性找到对应的参数，从而拿到对应的值

如在以上例子中，在 SQL 语句中使用了 `@Id` 标识，此时将可以通过前缀 `@` 判断取出需要替换的属性是 `Id` 属性。从对应的方法 `User GetById([Param("Id")]long id);` 的 `Param` 特性找到对应的参数是 `long id` 参数，因此在运行时将可以进行属性替换为参数

在 SmartSql 的属性前缀替换是十分智能的，将会根据所使用的数据库替换为不同的值，其目的是规避数据库的关键词以及合法变量。其次 SmartSql 也支持在业务端自定义属性前缀

默认定义如下

- SQL Server: `@`
- My SQL Server: `@`
- POSTGRESQL: `@`
- SQLite: `@`
- MySQL: `?`
- MySQL CONNECTOR: `?`
- ORACLE: `:`

以上默认定义在 [官方仓库的 src/SmartSql/DataSource/DbProviderManager.cs](https://github.com/dotnetcore/SmartSql/blob/e3a18b7c356b7a2033eaf3c9a55f9ca9b92121ec/src/SmartSql/DataSource/DbProviderManager.cs#L13-L66) 代码文件里

通过默认值定义可以了解到为什么从 SQLite 替换到 MySQL 时，可能存在 SQL 的属性替换失败，在 `SmartSql.Middlewares.PrepareStatementMiddleware` 的输出里面没有参数，显示代码是 `Parameters:[]` 没有参数

其中一个可能的原因是在 SQLite 中使用的 SQL 语句是 `Select * From T_User Where Id=@Id` 属性使用前缀是 `@` 符号，而通过默认值定义可以看到在 MySQL 的默认定义是 `?` 符号。因此需要将 SQL 语句替换为 `Select * From T_User Where Id=?Id` 才能替换参数

在使用方法上，因为如果 SQL 语句需要动态根据所使用的数据库而进行变更，那么在更换数据库时将会存在很大的工作量。因此 SmartSql 库提供了开发者自定义的属性前缀的方法，通过开发者自定义的属性前缀可以做到在更换数据库类型时，不需要更改 SQL 语句

在开发者端自定义属性前缀，可以在 SmartSqlMapConfig.xml 的 Settings 使用 ParameterPrefix 属性进行定义，如以下示例

```xml
<SmartSqlMapConfig xmlns="http://SmartSql.net/schemas/SmartSqlMapConfig.xsd">
    <Settings IgnoreParameterCase="false" ParameterPrefix="$" IsCacheEnabled="true" EnablePropertyChangedTrack="true"/>
</SmartSqlMapConfig>
```

以上代码将在开发者端定义属性前缀是 `$` 符号，此时在 SQL 语句中所有使用 `$` 开头的属性将会被识别为需要替换的属性，将会在运行时进行参数替换

在 SmartSql 的 SQL 属性替换的原理如下

在应用程序启动时，将创建 DbProviderManager.cs 的各个数据库对应的参数配置，将会初始化各个不同的数据库使用的默认属性前缀。详细代码请看 [官方仓库的 src/SmartSql/DataSource/DbProviderManager.cs](https://github.com/dotnetcore/SmartSql/blob/e3a18b7c356b7a2033eaf3c9a55f9ca9b92121ec/src/SmartSql/DataSource/DbProviderManager.cs#L13-L66) 代码

在启动过程的 `SmartSql.ConfigBuilder.XmlConfigBuilder.BuildDatabase()` 函数将根据配置文件决定使用哪个数据库，因此将拿到对应的数据库默认属性前缀

在 SmartSql 的设计里面，将在启动过程的 `SmartSql.ConfigBuilder.SqlMapBuilder.BuildStatements()` 读取所有的 XML 文件定义的 SQL 语句，在此方法进行构建

在 BuildStatements 方法构建的核心逻辑将会调用 `SmartSql.Configuration.Tags.TagBuilders.SqlTextBuilder.Build` 方法，此方法将会进行字符串替换，将 SQL 语句中所有用到自定义属性前缀的字符替换为具体数据库的默认属性前缀的值，代码如下

```csharp
        public ITag Build(XmlNode xmlNode, Statement statement)
        {
            var innerText = xmlNode.InnerText;
            var sqlConfig = statement.SqlMap.SmartSqlConfig;

            // 以下是核心代码，将替换对应的 SQL 语句中使用开发者自定义的前缀的字符替换为具体数据库的默认属性前缀的值
            var bodyText = innerText.Replace(sqlConfig.Settings.ParameterPrefix
                , sqlConfig.Database.DbProvider.ParameterPrefix);
            return new SqlText(bodyText
                , sqlConfig.Database.DbProvider.ParameterPrefix)
            {
                Statement = statement
            };
        }
```

以上代码的 innerText 是开发者编写的 SQL 语句，如 `Select * From T_User Where Id=$Id` 代码。而 `sqlConfig.Settings.ParameterPrefix` 对应在 SmartSqlMapConfig.xml 的 Settings 的 ParameterPrefix 属性。而 sqlConfig.Database.DbProvider.ParameterPrefix 是对应数据库的默认属性前缀的值

假定如上示例开发者自定义的属性前缀是 `$` 字符，而采用数据库是 `SQLite` 数据库，通过上文可以了解到 `SQLite` 数据库的默认属性前缀的值是 `@` 字符，因此以上代码等价于如下代码

```csharp
    var innerText = "Select * From T_User Where Id=$Id";
    var bodyText = innerText.Replace("$", "@");
```

因此最终的 SQL 语句都会使用对应数据库的默认属性前缀的值作为属性前缀

在运行的过程，在调用对应的访问数据库方法时，将会先找到对应的 SQL 语句，经过 [`SmartSql.Middlewares.PrepareStatementMiddleware.BuildDbParameters`](https://github.com/dotnetcore/SmartSql/blob/e3a18b7c356b7a2033eaf3c9a55f9ca9b92121ec/src/SmartSql/Middlewares/PrepareStatementMiddleware.cs#L65-L88) 方法进行属性替换为业务传入参数

其中这个步骤核心逻辑是将拿到的参数预先构建为 属性名-参数值 的字典，然后进入 [SqlParamAnalyzer 的 Replace](https://github.com/dotnetcore/SmartSql/blob/e3a18b7c356b7a2033eaf3c9a55f9ca9b92121ec/src/SmartSql/Utils/SqlParamAnalyzer.cs#L33) 方法进行属性和参数的替换逻辑

在 [SqlParamAnalyzer 的 Replace](https://github.com/dotnetcore/SmartSql/blob/e3a18b7c356b7a2033eaf3c9a55f9ca9b92121ec/src/SmartSql/Utils/SqlParamAnalyzer.cs#L33) 方法里面将通过正则替换的方法，找到 SQL 语句里面的各个属性，执行传入的属性和参数的替换方法，替换属性为对应的参数

因此如果想要让 SQL 语句能被正确替换属性，需要在 [SqlParamAnalyzer 的 Replace](https://github.com/dotnetcore/SmartSql/blob/e3a18b7c356b7a2033eaf3c9a55f9ca9b92121ec/src/SmartSql/Utils/SqlParamAnalyzer.cs#L33) 方法的第一步正则替换能正确执行。在 [SqlParamAnalyzer](https://github.com/dotnetcore/SmartSql/blob/e3a18b7c356b7a2033eaf3c9a55f9ca9b92121ec/src/SmartSql/Utils/SqlParamAnalyzer.cs) 的构造函数将会创建出正则，请看代码

```csharp
        public SqlParamAnalyzer(bool ignoreCase, string dbPrefix)
        {
            var regOptions = RegexOptions.Multiline | RegexOptions.CultureInvariant | RegexOptions.Compiled;
            if (ignoreCase)
            {
                regOptions = regOptions | RegexOptions.IgnoreCase;
            }

            _sqlParamsTokens = new Regex(@"[" + dbPrefix + @"]([\p{L}\p{N}_.\[\]]+)", regOptions);
        }
```

以上代码的 `dbPrefix` 将会被传入具体的数据库的默认属性前缀的值，如 `SQLite` 数据库的默认属性前缀的值是 `@` 字符

根据正则字符串可以找到 SQL 里面所有的属性字符串，将属性替换为具体的参数即可完成实际使用的 SQL 语句。输出的实际使用的 SQL 语句将会放在 RequestContext 的 RealSql 字符串中

在 appsettings.json 中将 LogLevel 的 Default 设置为 Debug 等级，可以在运行时看到从开发者编写的 SQL 语句加上参数的实际 SQL 语句，大概内容如下

```csharp
dbug: SmartSql.Middlewares.PrepareStatementMiddleware[0]
      Statement.Id:[User.GetEntity],Sql:
      Select * From T_User Where Id=@Id
      Parameters:[Id=1]
      Sql with parameter value:
      Select * From T_User Where Id=1
```

当前 SmartSql 的文档比较缺失，入门级文档请看 [smartsql 入门使用踩坑笔记 - J.晒太阳的猫 - 博客园](https://www.cnblogs.com/jasongrass/p/13656356.html )






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。