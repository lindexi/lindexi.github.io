# dotnet 警惕 Assembly.Location 返回空

在大部分情况下，获取当前所运行的应用程序的所在路径时，常用的就是 Assembly.Location 属性，按照之前的经验，使用 Assembly.Location 是最为稳定的做法，然而在 dotnet 发布单文件时，此属性将会为空，导致一些不符合预期的行为

<!--more-->
<!-- CreateTime:2023/10/16 19:53:54 -->

<!-- 博客 -->
<!-- 发布 -->

通过 Assembly.Location 属性可以返回程序集所在的文件路径，这个一个比较稳定的获取某个路径方式，至少比获取当前的工作路径 `Environment.CurrentDirectory` 和 `Directory.GetCurrentDirectory()` 都要稳定得多。每次使用 Assembly.Location 都是返回程序集所在的文件路径，而工作路径 `Environment.CurrentDirectory` 和 `Directory.GetCurrentDirectory()` 则是返回当前的工作路径，而大家都知道，工作路径是可以非常简单的被进行更改的，从而导致每次调用 `Environment.CurrentDirectory` 或 `Directory.GetCurrentDirectory()` 都可以返回不同的值。当然了，是否使用工作路径，这也是看大家的需求的

如果大家在阅读以上内容时，还没有工作路径的概念，还请先自行了解一下工作路径是什么以及工作路径的用途是什么

在单文件发布这个功能之前，当咱需要获取当前的应用程序安装路径，在不考虑插件 DLL 存在的情况下，我是推荐使用 Assembly.Location 属性获取当前的应用程序所在的文件夹的，大概的代码如下

```csharp
string installFolder = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location)!;
```

这里无论是采用 GetExecutingAssembly 也好，还是 GetEntryAssembly 方法都对于应用程序来说是正确的。但是由于单元测试下是没有入口的程序集的也就是 GetEntryAssembly 将返回空，于是此时换成 GetExecutingAssembly 获取当前正在运行的代码所在的程序集将是更加稳定的

通过以上方式获取应用程序路径将比使用 AppDomainSetup.ApplicationBase 更加稳定，如以下代码是通过 AppDomainSetup.ApplicationBase 获取路径

```csharp
// 以下代码是不推荐的
string? installFolder = AppDomain.CurrentDomain.SetupInformation.ApplicationBase;
```

然而原本比较稳定的 Assembly.Location 属性将在进行单文件发布时，返回空字符串。这就让许多现有的逻辑不能正常工作，好在发布单文件时，将会看到 VisualStudio 的以下提示内容

```
IL3000: Avoid accessing Assembly file path when publishing as a single file
```

这时候的推荐使用的是 `AppContext.BaseDirectory` 属性，这个属性也是用来返回当前应用程序的安装路径的稳定属性

换句话说就是在使用 dotnet core 时，无论是 .NET Core 3.1 还是 dotnet 6 版本，在获取当前应用程序的安装路径时，都可以使用 `AppContext.BaseDirectory` 属性。使用这个属性不仅代码短，且稳定

那此时就有伙伴会疑惑，为什么我之前推荐都是使用 `Assembly.Location` 属性。这是因为我的许多基础库和项目那会都需要兼容 .NET Framework 4.5 版本，而 `AppContext.BaseDirectory` 是在 .NET Framework 4.6 之后才引入的，这就是为什么我没有推荐过这个属性的原因

如果自己的项目里面有大量的旧代码都是采用 `Assembly.Location` 属性，感觉改不动，或者是在基础库里面就是采用 `Assembly.Location` 属性的，那可以使用配置方式切换为兼容逻辑，如下面代码

```csharp
AppContext.SetSwitch("Switch.System.Reflection.Assembly.SimulatedLocationInBaseDirectory", true);
```

以上配置推荐加在 Main 函数第一句话里面，加上以上配置之后，即可让 `Assembly.Location` 属性返回的是当前单文件的路径，而不会返回空字符串

以上的配置内容是在 https://github.com/dotnet/corert/issues/5467 里面大佬提供的

更多博客内容请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html ) 或 [博客园的合集](https://www.cnblogs.com/lindexi/collections/6439)