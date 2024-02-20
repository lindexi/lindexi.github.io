# dotnet 禁用 SQLite 的 SQLiteFunction 扫描程序集提升启动性能

在我所在的团队开发的一个 WPF 应用程序里面，使用到了 SQLite 作为本地数据库。在优化启动性能过程中，发现了在启动过程一旦访问 SQLite 将会因为 SQLiteFunction 扫描程序集导致 CPU 损耗，从而影响启动性能。本文将告诉大家如何禁用 SQLite 的 SQLiteFunction 扫描程序集

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

在 SQLiteFunction 模块里面，将会扫描全部程序集，用来找到 SQLiteFunctionAttribute 实现类型收集。刚好我的应用不需要这样的功能，这就意味着在 SQLiteFunction 模块里面扫描全部程序集的逻辑是白跑的，禁用此逻辑可提升启动性能

禁用方法可以是在 Main 方法里面设置环境变量的方式实现禁用 SQLiteFunction 模块扫描全部程序集。以下代码需要放在应用程序运行足够早的时间，至少需要比首个 SQLite 相关模块早，由于以下代码仅仅只是设置环境变量，性能损耗上很小，可以比较放心的在 Main 方法里面设置

```csharp
       // 在 SQLite 的 SQLiteFunction 类的静态构造函数会反射扫一遍所有程序集，找 SQLiteFunctionAttribute 特性
       // 我们不需要这个功能，通过配置这个环境变量，避免扫描，优化启动时 CPU 占用
       Environment.SetEnvironmentVariable("No_SQLiteFunctions", "1");
```

特别感谢 [lsj](https://blog.sdlsj.net/ ) 提供的方法，我只是代为记录

以上环境变量请参阅 [SQLite Environment Variables](https://system.data.sqlite.org/index.html/doc/216889c23b/Doc/Extra/Provider/environment.html ) 官方文档