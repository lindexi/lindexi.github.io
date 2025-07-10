# dotnet 测试 AOT 的 API 行为

本文记录我测试 AOT 的 API 行为

<!--more-->
<!-- CreateTime:2025/07/10 07:09:23 -->

<!-- 发布 -->
<!-- 博客 -->

判断当前代码正在 AOT 模式下运行：

```csharp
RuntimeFeature.IsDynamicCodeSupported
```

获取入口程序集 `Assembly.GetEntryAssembly()` 返回非空，返回真的入口程序集。此行为和单元测试下不同，单元测试下返回空

获取调用当前方法的程序集 `Assembly.GetCallingAssembly` 可能抛出 PlatformNotSupportedException 异常

获取正在执行的程序集 `Assembly.GetExecutingAssembly` 返回非空

测试代码如下

```csharp
using System.Diagnostics;
using System.Reflection;
using System.Runtime.CompilerServices;

Console.WriteLine($"RuntimeFeature.IsDynamicCodeSupported={RuntimeFeature.IsDynamicCodeSupported}");

Console.WriteLine($"Assembly.GetEntryAssembly() is not null={Assembly.GetEntryAssembly() is not null}");
try
{
    Console.WriteLine($"Assembly.GetCallingAssembly() is not null={Assembly.GetCallingAssembly() is not null}");
}
catch (Exception e)
{
    Console.WriteLine($"Assembly.GetCallingAssembly() Exception. {e.GetType().FullName}: {e.Message}");
}

Console.WriteLine($"Assembly.GetExecutingAssembly() is not null={Assembly.GetExecutingAssembly() is not null}");

Console.WriteLine($"Assembly.GetEntryAssembly()?.GetCustomAttribute<DebuggableAttribute>() is not null={Assembly.GetEntryAssembly()?.GetCustomAttribute<DebuggableAttribute>() is not null}");
```

以上代码的 AOT 输出内容如下

```
RuntimeFeature.IsDynamicCodeSupported=False
Assembly.GetEntryAssembly() is not null=True
Assembly.GetCallingAssembly() Exception. System.PlatformNotSupportedException: Operation is not supported on this platform.
Assembly.GetExecutingAssembly() is not null=True
Assembly.GetEntryAssembly()?.GetCustomAttribute<DebuggableAttribute>() is not null=False
```

本文以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0295a1d5ac9c60903fe0674859b83d421bbf1a08/Workbench/NufubijemlerWabearnagaqem) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/0295a1d5ac9c60903fe0674859b83d421bbf1a08/Workbench/NufubijemlerWabearnagaqem) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0295a1d5ac9c60903fe0674859b83d421bbf1a08
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0295a1d5ac9c60903fe0674859b83d421bbf1a08
```

获取代码之后，进入 Workbench/NufubijemlerWabearnagaqem 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )