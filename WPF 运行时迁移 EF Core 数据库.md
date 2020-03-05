# WPF 运行时迁移 EF Core 数据库

在客户端开发，可以使用 .NET Core 3.0 开发 WPF 程序，可以使用 EF Core 连接数据库。客户端的数据库使用 SQLite 在不同的版本需要在客户端运行做数据库迁移升级数据库

<!--more-->
<!-- CreateTime:2019/12/19 8:56:35 -->

<!-- csdn -->

在 WPF 使用 EF Core 可以安装下面的库

```csharp
  <PackageReference Include="Microsoft.AspNetCore.Mvc.NewtonsoftJson" Version="3.0.0" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="3.0.0">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
  </PackageReference>
  <PackageReference Include="Microsoft.EntityFrameworkCore" Version="3.0.0"/>
  <PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="3.0.0" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="3.0.0" />
```

先创建一个 Model 类，在这个类里面需要添加 Id 属性，如创建资源类

```csharp
    public class ResourceModel
    {
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public string Id { set; get; }

        public string ResourceId { set; get; }

        public string ResourceName { set; get; }

        public string LocalPath { set; get; }

        public string ResourceSign { set; get; }

        public string ResourceFileDetail { set; get; }
    }
```

然后创建数据类，用于连接数据库

```csharp
    public class KekairwuceeYernellijewhebere : DbContext
    {
        public DbSet<ResourceModel> ResourceModel { get; set; }

        /// <inheritdoc />
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            var file = Path.Combine("FileManger.db");
            file = Path.GetFullPath(file);
            optionsBuilder
                .UseSqlite($"Filename={file}");
        }
    }
```

重写 OnConfiguring 方法在里面写连接方法，此时就完成了数据定义，但是还没有创建数据库

使用命令行进行数据库迁移，数据库迁移就是创建数据库相关代码，在第一次进行迁移将会自动创建代码用于创建数据库

```csharp
dotnet ef migrations add 版本名 
```

上面代码的版本名可以随意命名，如我是这样写

```csharp
dotnet ef migrations add Lindexi 
```

执行上面代码可以看到在项目里面添加了 Migrations 文件夹，这个文件夹里面包含数据库的迁移代码

在主函数可以使用下面代码创建数据库，如果数据库已经创建了那么将什么都不做

```csharp
            using (var kekairwuceeYernellijewhebere = new KekairwuceeYernellijewhebere())
            {
                kekairwuceeYernellijewhebere.Database.Migrate();
            }
```

如果只是一次性创建，之后不执行修改的，可以使用 EnsureCreated 函数创建，请看下面代码

```csharp
            using (var kekairwuceeYernellijewhebere = new KekairwuceeYernellijewhebere())
            {
                kekairwuceeYernellijewhebere.Database.EnsureCreated();
            }
```

注意使用 EnsureCreated 函数创建之后，将在下次调用 Database.Migrate 函数时提示下面代码

```
Microsoft.Data.Sqlite.SqliteException:“SQLite Error 1: 'table "ResourceModel" already exists'.”
```

如果软件更新了，需要修改 ResourceModel 的内容，添加一个属性

```csharp
   public class ResourceModel
    {
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public string Id { set; get; }

        public string ResourceId { set; get; }

+       public string WaircegalhallwayneeHuwairfejaije { set; get; }

        public string ResourceName { set; get; }

        public string LocalPath { set; get; }

        public string ResourceSign { set; get; }

        public string ResourceFileDetail { set; get; }
    }
```

那么在修改代码之后，再次执行迁移命令

```csharp
dotnet ef migrations add 版本名 
```

此时建议创建迁移代码，在软件运行的时候执行 Migrate 函数将会自动升级数据库

如果数据库是需要升级的，那么请使用 Database.Migrate 函数创建数据库，之后可以在访问数据库之前调用这个函数让数据库如果没有更新就自动更新

每次调用 Migrate 都需要一定的时间，建议在另一个线程运行

如果在运行 SaveChanges 提示 `no such table` 那么可能是在调用 `Migrate` 等方法之前没有先调用 `dotnet ef migrations` 创建迁移类

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
