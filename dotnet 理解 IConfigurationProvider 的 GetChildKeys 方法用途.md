# dotnet 理解 IConfigurationProvider 的 GetChildKeys 方法用途

我最近遇到一个有趣的 Bug 让我调试了一会，其现象是我的好多个模块都因为读取不到配置信息而炸掉。核心原因是我对 IConfigurationProvider 的 GetChildKeys 方法的理解不对，实现错了此方法。本文将告诉大家 IConfigurationProvider 的 GetChildKeys 方法用途和如何正确实现他

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

在开始之前，先感谢两位大佬的博客：

- [理解ASP.NET Core - 配置(Configuration) - xiaoxiaotank - 博客园](https://www.cnblogs.com/xiaoxiaotank/archive/2021/10/08/15367747.html )
- [.NET Core 3.0之深入源码理解Configuration(一) - 艾心 - 博客园](https://www.cnblogs.com/edison0621/p/10854215.html )

要不是有这两篇博客，我还没有反应过来是我对 GetChildKeys 的理解不对

故事是： 我在使用 [COIN 配置库](https://github.com/dotnet-campus/dotnetCampus.Configurations/)对接 Microsoft.Extensions.Configuration 的时候，我需要写一个中间类型，让这个中间类型对接 COIN 和 Microsoft.Extensions.Configuration 配置。这个中间类型就需要实现 IConfigurationSource 和 IConfigurationProvider 接口

这个 [COIN 配置库](https://github.com/dotnet-campus/dotnetCampus.Configurations/)是我所在的团队开源的高性能配置库，主打稳定和高性能，特别适合客户端应用做对接配置文件使用，详细请看 [https://github.com/dotnet-campus/dotnetCampus.Configurations/](https://github.com/dotnet-campus/dotnetCampus.Configurations/)

对接的时候，我将此中间的类型称为 ReadonlyCoinConfiguration 类型，此中间的类型就是从 COIN 配置里面读取出配置，提供只读的方式，不允许将配置设置回 COIN 配置

让 ReadonlyCoinConfiguration 类型同时继承 IConfigurationSource 和 IConfigurationProvider 类型，从而可以所有代码都写到一个类型里面

先实现 IConfigurationSource 的 Build 方法，返回自身

```csharp
class ReadonlyCoinConfiguration : IConfigurationSource, IConfigurationProvider
{
    public IConfigurationProvider Build(IConfigurationBuilder builder)
    {
        return this;
    }

    // 忽略其他代码
}
```

接下来就是对 IConfigurationProvider 的实现。先集中到本文的主题，也就是 GetChildKeys 函数上。我出现问题的代码是采用如下定义

```csharp
    public IEnumerable<string> GetChildKeys(IEnumerable<string> earlierKeys, string parentPath)
    {
        return Array.Empty<string>();
    }
```

以上的代码的实现是不符合预期的，如果我在配置初始化完成之后，在业务端调用 Configuration 的 AsEnumerable() 方法，将拿到空列表

```csharp
        var builder = WebApplication.CreateBuilder(args);

        builder.Configuration.AddInMemoryCollection().Add(new ReadonlyCoinConfiguration());

        builder.Services.AddControllers();

        var app = builder.Build();

        var keyValuePairs = app.Configuration.AsEnumerable().ToList();
```

以上代码的 keyValuePairs 的元素是 0 个

在框架里面，设计的 GetChildKeys 函数的功能是有两个方面考虑：

1. 对其他的 IConfigurationProvider 的结果进行过滤
2. 返回给框架层，此 IConfigurationProvider 提供的配置项

在 Microsoft.Extensions.Configuration 里是支持多个配置的，也就是支持多个 IConfigurationProvider 同时工作，且后加入的 IConfigurationProvider 的优先级或者说是权重更高的。例如 Microsoft.Extensions.Configuration 里同时传入 JSON 和 XML 和 Ini 和命令行作为配置，且命令行的配置期望是高优先级的。也就是在命令行里面的配置可以覆盖其他的配置信息

另外，由于一些业务是对配置项的顺序是敏感的，也就是配置项的顺序是会影响业务的逻辑的。例如配置里面有 Foo1 和 Foo2 这两项，在获取所有配置的时候，如果返回的顺序是 Foo1 在先然后是 Foo2 的顺序，和 Foo2 在先然后是 Foo1 的顺序也许将会影响业务的执行逻辑。于是在 Microsoft.Extensions.Configuration 里也期望能够让 IConfigurationProvider 可以控制配置项的顺序

在这两个需求的前提下，就设计了 GetChildKeys 方法

在 GetChildKeys 方法传入的两个参数的含义分别是：

- earlierKeys： 在此 IConfigurationProvider 之前的其他的 IConfigurationProvider 提供的配置项
- parentPath： 当前期望获取的配置的前缀路径，例如 `Logging:Microsoft.Extensions.Logging.Console.ConsoleLoggerProvider:FormatterOptions` 前缀等

返回值是期望获取到可供输出的配置项。可供输出的意思就是将传入的 `earlierKeys` 其他 IConfigurationProvider 提供的配置项，再加上本 IConfigurationProvider 提供的配置项组合过滤之后的配置项列表

也就是说如果我需要在 IConfigurationProvider 实现过滤某些配置项的功能，那我只需要在返回的时候，将 `earlierKeys` 进行过滤之后返回即可

如果我只是期望追加一些新的配置，那我只需要将我的新的配置追加到 `earlierKeys` 一起返回即可。这是比较常用的方法，通过 Concat 的方式配合组装为 IEnumerable 返回，如下面代码，追加了三个配置项

```csharp
    public IEnumerable<string> GetChildKeys(IEnumerable<string> earlierKeys, string parentPath)
    {
        return new string[] { "Foo.F1", "Foo.F2", "Foo.F3" }.Concat(earlierKeys);
    }
```

如果我是期望返回的时候进行排序，那我只需要在最终返回之前进行排序即可

另外，在追加的时候，也是包含顺序的，例如上面代码的追加，最终拿到的配置项列表大概如下

```
            [WTTSTDIO, C:\Program Files (x86)\Windows Kits\10\Hardware Lab Kit\Studio\]
            [windir, C:\Windows]
            ...
            [APPDATA, C:\Users\lindexi\AppData\Roaming]
            [ALLUSERSPROFILE, C:\ProgramData]
            [AllowedHosts, *]
            [, ]
            [:ASPNETCORE_BROWSER_TOOLS, true]
            [:Foo.F3, ]
            [:Foo.F2, ]
            [:Foo.F1, ]
            [Foo.F3, ]
            [Foo.F2, ]
            [Foo.F1, ]
```

而如果我是这样写的：

```csharp
return earlierKeys.Concat(new string[] { "Foo.F1", "Foo.F2", "Foo.F3" });
```

那么拿到的配置列表大概如下

```
          [Foo.F3, ]
          [Foo.F2, ]
          [Foo.F1, ]
          [WTTSTDIO, C:\Program Files (x86)\Windows Kits\10\Hardware Lab Kit\Studio\]
          [windir, C:\Windows]
          ...
          [APPDATA, C:\Users\lindexi\AppData\Roaming]
          [ALLUSERSPROFILE, C:\ProgramData]
          [AllowedHosts, *]
          [, ]
          [:Foo.F3, ]
          [:Foo.F2, ]
          [:Foo.F1, ]
          [:ASPNETCORE_BROWSER_TOOLS, true]
```

这里有一点需要注意的是，返回的时候，需要根据 `parentPath` 的内容，过滤掉当前 IConfigurationProvider 能提供的配置，才能和 earlierKeys 组合后返回

```csharp
    public IEnumerable<string> GetChildKeys(IEnumerable<string> earlierKeys, string parentPath)
    {
        // 这个方法的作用其实有两个：
        // 1. 对其他的 IConfigurationProvider 的结果进行过滤
        // 2. 返回给框架层，此 IConfigurationProvider 提供的配置项

        // 自己测试：
        // 1. 什么都不做，返回的是 earlierKeys 的内容
        // 2. 直接返回 Array.Empty<string>();
        // 3. 拼接出新的列表，如 

        // 例如这个类型提供的配置里面，包含的是 Foo.F1=123; Foo.F2=123; Foo.F3=123 三个值内容
        // 传入的 父路径(parentPath) 是叫做 `Foo` 那么就应该将 `Foo.F1` 和 `Foo.F2` 和 `Foo.F3` 三个 Key 项合并 earlierKeys 进行返回
        // 默认都是采用 Concat(earlierKeys) 的方式进行返回的
        // 那什么情况下不是采用直接 Concat(earlierKeys) 的方式？嗯，需要过滤掉，或者是需要进行重新排序

        if (string.IsNullOrEmpty(parentPath)) // 加上一些判断逻辑
        {
            /*
            [WTTSTDIO, C:\Program Files (x86)\Windows Kits\10\Hardware Lab Kit\Studio\]
            [windir, C:\Windows]
            ...
            [APPDATA, C:\Users\lindexi\AppData\Roaming]
            [ALLUSERSPROFILE, C:\ProgramData]
            [AllowedHosts, *]
            [, ]
            [:ASPNETCORE_BROWSER_TOOLS, true]
            [:Foo.F3, ]
            [:Foo.F2, ]
            [:Foo.F1, ]
            [Foo.F3, ]
            [Foo.F2, ]
            [Foo.F1, ]
             */
            return new string[] { "Foo.F1", "Foo.F2", "Foo.F3" }.Concat(earlierKeys);
          
        }

        return earlierKeys;
    }
```

在大部分的情况下的返回值都是判断 `parentPath` 参数，通过此参数过滤当前的 IConfigurationProvider 能提供的配置。再使用 Concat 方法，和 `earlierKeys` 组合后返回

以及在某些情况下，将 `earlierKeys` 给进行一次过滤或排序之后再返回。如下面代码，对 `earlierKeys` 进行 Where 之后再一次组合

```csharp
return new string[] { "Foo.F1", "Foo.F2", "Foo.F3" }.Concat(earlierKeys.Where(t => !t.StartsWith("Foo")))
```

换句话说就是，大部分时候传入的 `earlierKeys` 参数是需要在返回值返回的，或者是参与了一定的计算之后再返回，而不是吞掉，直接返回一个自定义的列表

如果和本文开始的方法一样，返回了 `Array.Empty<string>()` 那就意味着在这个 IConfigurationProvider 里面，提供的功能是将所有的配置项都给过滤掉。于是各个需要枚举所有配置内容的业务都会影响找不到期望的配置而炸掉

可以看到 IConfigurationProvider 的 GetChildKeys 方法还是很强大的。从定义的设计上，既满足了支持过滤其他的 IConfigurationProvider 提供的配置，又支持加入自身的定制的配置。同时依靠 dotnet 提供的强大的 IEnumerable 能力，可以做到无大内存空间分配。如上面代码的 Concat 和 Where 等，本质都是延迟执行且无需重新申请数组空间，这部分知识详细请自行了解 dotnet 基础知识

另外，如果只是纯粹想多添加一些新的配置到应用，除了直接继承 IConfigurationProvider 之外，还可以继承 ConfigurationProvider 类型。继承 ConfigurationProvider 类型之后，可以给他添加新的配置，其他的琐杂的工作就都交给 ConfigurationProvider 处理

```csharp
class ReadonlyCoinConfiguration : ConfigurationProvider, IConfigurationSource
{
    public IConfigurationProvider Build(IConfigurationBuilder builder)
    {
        return this;
    }

    public override void Load()
    {
        Set("Foo.F1", "123");
        Set("Foo.F2", "123");
        Set("Foo.F3", "123");
    }
}
```

如以上的代码，可以看到，只需要重写 Load 方法，在此方法里面，将所能提供的配置项调用 Set 方法写入即可