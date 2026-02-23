# dotnet 在新进程执行某段委托的方法

在一些测试逻辑中，经常需要有简单的逻辑期望在新进程里跑。为了不让这些逻辑每次都需要新建一个项目，我就期望做一个简单的库，通过这个库支持传入一段在新进程里面执行的委托给新进程去执行

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

大概的 API 设计如下：

```csharp
RemoteExecutor.Invoke(() =>
{
    // 在这里编写在新进程执行的委托代码
});
```

要在 Main 函数里面调用 `RemoteExecutor.TryHandle` 处理命令行，因为新进程里面执行的逻辑本身就需要 Main 函数参与。标准预期写法如下

```csharp
if (RemoteExecutor.TryHandle(args))
{
    return;
}
```

核心实现原理就是反射获取委托所在的方法，通过方法反射调用而已

大家都知道，在 C# dotnet 里面，委托是会被生成为独立类型的。利用此原理，获取委托所在的程序集、类型、方法名，将其作为命令行参数传递过去到新进程。在新进程里面，读取传入的命令行参数，了解到当前应该反射执行哪个方法，从而执行到委托的逻辑

这个过程里面，可以看到是给新进程去执行的，意味着过程中禁止任何捕获字段，任何的委托捕获对象都不能给传递到新进程里面

以下是示例调用的全部代码：

```csharp
using System.Globalization;
using RemoteExecutors;

if (RemoteExecutor.TryHandle(args))
{
    return;
}

RemoteExecutor.Invoke(() =>
{
    // 写个文件测试一下
    var file = Path.Join(AppContext.BaseDirectory, "1.txt");
    File.WriteAllText(file, DateTime.Now.ToString(CultureInfo.InvariantCulture));
});

Console.WriteLine("Hello, World!");
```

可以看到在这个写法里面，可以很方便将一个委托放在另一个进程去执行

本文提供的 RemoteExecutor 类的实现也非常简单，大家看一下代码就明白了原理

```csharp
public static class RemoteExecutor
{
    public static void Invoke(Action action)
    {
        var method = action.Method;

        Type? type = method.DeclaringType;

        if (type is null)
        {
            throw new ArgumentException();
        }

        TypeInfo typeInfo = IntrospectionExtensions.GetTypeInfo(type);

        var methodName = method.Name;
        var className = typeInfo.FullName!;
        var assemblyFullName = typeInfo.Assembly.FullName!;

        string[] commandLineArgs = 
            [
                RemoteExecutorOption.CommandName,
                "--AssemblyName", assemblyFullName,
                "--ClassName", className,
                "--MethodName", methodName,
            ];

        var processPath = Environment.ProcessPath;
        if (processPath is null)
        {
            throw new InvalidOperationException();
        }

        var process = Process.Start(processPath,commandLineArgs);
        process.WaitForExit();
    }

    public static bool TryHandle(string[] commandLineArgs)
    {
        var index = commandLineArgs.IndexOf(RemoteExecutorOption.CommandName);
        if (index == -1)
        {
            return false;
        }

        var optionCommandLineArgs = commandLineArgs.Skip(index+1).ToList();
        var result = CommandLine.Parse(optionCommandLineArgs)
            .AddHandler<RemoteExecutorOption>(option =>
            {
                var assemblyName = option.AssemblyName;
                var className = option.ClassName;
                var methodName = option.MethodName;

                var assembly = Assembly.Load(assemblyName);
                var classType = assembly.GetType(className)!;

                var methodInfo = classType.GetTypeInfo().GetDeclaredMethod(methodName)!;
                object? instance = null;
                if (!methodInfo.IsStatic)
                {
                    instance = Activator.CreateInstance(classType);
                }
                object? result = methodInfo.Invoke(instance, null);
                _ = result;
            })
            .Run();

        _ = result;

        return true;
    }
}

internal class RemoteExecutorOption
{
    public const string CommandName = "RemoteExecutor_F6679170-3719-49AB-9936-7CAB5AB6294D";

    [Option]
    public required string AssemblyName { get; init; }

    [Option]
    public required string ClassName { get; init; }

    [Option]
    public required string MethodName { get; init; }
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/a3a10076765bea695136442c8745d18b42d840f2/Workbench/DurwerjeguCalldemrereher) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/a3a10076765bea695136442c8745d18b42d840f2/Workbench/DurwerjeguCalldemrereher) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin a3a10076765bea695136442c8745d18b42d840f2
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin a3a10076765bea695136442c8745d18b42d840f2
```

获取代码之后，进入 Workbench/DurwerjeguCalldemrereher 文件夹，即可获取到源代码