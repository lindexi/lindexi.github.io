
# dotnet 通过引用 msbuild 程序集实现自己定制编译器

本来我想说的是基于引用 msbuild 程序集来自己做一个编译器，但是想想好像本文做的，和造编译器没啥关系，咱自己调用 msbuild 的 API 而已。本文来告诉大家如何引用 msbuild 程序集，如何在自己的应用程序里面嵌入 msbuild 的构建代码，实现 dotnet build 的效果

<!--more-->


<!-- 发布 -->

大部分的代码都是采用命令行的方式去调用 dotnet build 或 msbuild 命令，然而通过命令行调用用的是跨进程的方式，如果期望做更多的定制化，最好还是放在相同的进程，此时可以更改构建的各个步骤

自己制作一个编译器最简单的方法就是引用现有的成熟的编译器作为组件，刚好 msbuild 最新版本也是使用 dotnet 框架编写的，咱的 dotnet 应用可以非常方便将 msbuild 引用进来。当然了，本文不讨论如何自己发布 msbuild 的问题，因为这又是另一个坑了。本文的方法是引用本机已安装好的 msbuild 程序集

在开始之前，请新建一个控制台项目。当然了，你要是新建一个 WPF 项目也没啥问题

编辑 csproj 文件，添加如下代码

```xml
  <ItemGroup>
    <PackageReference Include="Microsoft.Build" Version="16.10.0" ExcludeAssets="runtime" />
    <PackageReference Include="Microsoft.Build.Utilities.Core" Version="16.10.0" ExcludeAssets="runtime" />
    <PackageReference Include="Microsoft.Build.Locator" Version="1.4.1" />
  </ItemGroup>
```

添加完成之后的 csproj 文件代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.Build" Version="16.10.0" ExcludeAssets="runtime" />
    <PackageReference Include="Microsoft.Build.Utilities.Core" Version="16.10.0" ExcludeAssets="runtime" />
    <PackageReference Include="Microsoft.Build.Locator" Version="1.4.1" />
  </ItemGroup>
</Project>
```

第一步是先获取本机已安装好的 msbuild 实例，如下代码

```csharp
        static void Main(string[] args)
        {
            var instances = MSBuildLocator.QueryVisualStudioInstances().ToList();
        }
```

以上代码要能运行，需要加上如下命名空间

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Microsoft.Build.Construction;
using Microsoft.Build.Evaluation;
using Microsoft.Build.Framework;
using Microsoft.Build.Locator;
```

以上拿到的 instances 就是本机安装的 msbuild 实例，也就是 dotnet sdk 的各个版本，可以使用如下代码输出

```csharp
            for (var i = 1; i <= instances.Count; i++)
            {
                var instance = instances[i - 1];
                var recommended = string.Empty;

                // The dev console is probably always the right choice because the user explicitly opened
                // one associated with a Visual Studio install. It will always be first in the list.
                if (instance.DiscoveryType == DiscoveryType.DeveloperConsole)
                    recommended = " (Recommended!)";

                Console.WriteLine($"{i}) {instance.Name} - {instance.Version}{recommended}");
            }
```

下一步是有点黑科技的部分，也就是为什么我会写本文的原因。使用下面代码注册 msbuild 实例，如果没有使用下面这句代码注册，那么在后续调用 msbuild 相关类型时，将会因为找不到 msbuild 的程序集而失败

```csharp
            // 必须调用 RegisterInstance 方法，否则将提示找不到 msbuild 文件
            MSBuildLocator.RegisterInstance(instances.First());
```

注册完成之后，将可以使用 msbuild 提供的各个类来实现构建，请新建一个方法用来编写调用 msbuild 各个类的构建代码。如以下代码

```csharp
        private static void Build()
        {
            var projectFile = new FileInfo(@"..\..\..\RalboleaballNeeqaqiku.csproj");

            var projectRootElement = ProjectRootElement.Open(projectFile.FullName);
            var project = new Project(projectRootElement);
            project.Build(new Logger());
        }
```

为什么需要这部分构建代码放在另一个方法里面？原因是在碰到了 ProjectRootElement 类型的时候，就需要开始加载程序集，然而在调用 MSBuildLocator.RegisterInstance 之前，还是找不到程序集的哦。因此为了让 MSBuildLocator.RegisterInstance 能被执行，就需要让包含 MSBuildLocator.RegisterInstance 代码的方法不会在执行之前碰到还没有存在的程序集，因此就需要将碰到构建相关逻辑的代码放在独立的方法或者独立的类型里面，这样就能让包含 MSBuildLocator.RegisterInstance 的代码不会因为找不到程序集而不执行

以上代码是通过调用 ProjectRootElement.Open 方法加载了 csproj 文件，此步骤是反序列化过程。接着新建 Project 实例，在新建方法里面将会进行初始化，可以拿到输入的 csproj 将有哪些导入等信息

最后一步是通过调用 Project 的 Build 方法进行构建，此时将会执行一次构建，构建的信息通过传入的 Logger 进行输出，以下是 Logger 的代码

```csharp
        private class Logger : ILogger
        {
            public void Initialize(IEventSource eventSource)
            {
                eventSource.AnyEventRaised += (_, args) => { Console.WriteLine(args.Message); };
            }

            public void Shutdown()
            {
            }

            public LoggerVerbosity Verbosity { get; set; }
            public string Parameters { get; set; }
        }
```

全部代码如下

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Microsoft.Build.Construction;
using Microsoft.Build.Evaluation;
using Microsoft.Build.Framework;
using Microsoft.Build.Locator;

namespace RalboleaballNeeqaqiku
{
    class Program
    {
        static void Main(string[] args)
        {
            var instances = MSBuildLocator.QueryVisualStudioInstances().ToList();

            for (var i = 1; i <= instances.Count; i++)
            {
                var instance = instances[i - 1];
                var recommended = string.Empty;

                // The dev console is probably always the right choice because the user explicitly opened
                // one associated with a Visual Studio install. It will always be first in the list.
                if (instance.DiscoveryType == DiscoveryType.DeveloperConsole)
                    recommended = " (Recommended!)";

                Console.WriteLine($"{i}) {instance.Name} - {instance.Version}{recommended}");
            }

            // 必须调用 RegisterInstance 方法，否则将提示找不到 msbuild 文件
            MSBuildLocator.RegisterInstance(instances.First());

            // 需要将构建的代码放在另一个方法里面，否则将会因为放在相同的方法，没有加上程序集
            Build();
        }

        private static void Build()
        {
            var projectFile = new FileInfo(@"..\..\..\RalboleaballNeeqaqiku.csproj");

            var projectRootElement = ProjectRootElement.Open(projectFile.FullName);
            var project = new Project(projectRootElement);
            project.Build(new Logger());
        }

        private class Logger : ILogger
        {
            public void Initialize(IEventSource eventSource)
            {
                eventSource.AnyEventRaised += (_, args) => { Console.WriteLine(args.Message); };
            }

            public void Shutdown()
            {
            }

            public LoggerVerbosity Verbosity { get; set; }
            public string Parameters { get; set; }
        }
    }
}
```

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b6171297d4200586d135a8c5c0d7376df7ee7c6a/RalboleaballNeeqaqiku) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b6171297d4200586d135a8c5c0d7376df7ee7c6a/RalboleaballNeeqaqiku) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b6171297d4200586d135a8c5c0d7376df7ee7c6a
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 RalboleaballNeeqaqiku 文件夹

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。