
# dotnet 使用 DotNetHost.Win32Metadata 让 .NET Framework 与 Native AOT 程序加载 .NET 10 组件

本文介绍如何用 wherewhere 大佬开源的 DotNetHost.Win32Metadata 库，在 C# 里直接调用 hostfxr 的原生 hosting API，让 .NET Framework 4.8.1 宿主和 Native AOT 宿主都能加载并运行 .NET 10 的托管组件

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

本文内容由人类主导 AI 辅助编写

## 背景

平时在 C# 里加载别的程序集，一个 `Assembly.LoadFrom` 就够了。但如果宿主和组件压根不是同一个 .NET 运行时，引用程序集这条路就走不通。我碰到的具体场景有这么几个：

- 想要 Native AOT 的启动速度，但又想要反射、插件加载、动态执行代码的能力。而这两件事在 Native AOT 里是天然冲突的

- 想要在一个 .NET Framework 4.8.1 的老程序里，跑一段最新的 .NET 10 的代码

- 想要让程序自己决定入口，比如同一个安装包里放多个版本的业务程序集，由启动器决定这次用哪一个

这三件事看起来毫不相干，但解法是同一个：不要试图引用另一个运行时，而是把另一个运行时拉起来。在 Windows 上把 .NET 运行时拉起来，微软给的官方做法就是 hostfxr

问题是，现在没有一个现成的官方方案能让我在 C# 里直接用这套东西。要在 C# 里用它，只能自己去读 `nethost.h`、`hostfxr.h`、`coreclr_delegates.h` 这几个头文件，把里面的 `typedef`、结构体、枚举一条条翻译成 C# 的 `delegate`、`struct`、`enum`，再配上几十个 `DllImport`

翻译本身不是不能做，只是有些体力活。想找找有哪位大佬干了这个活，造福大家

我找到的方案就是 wherewhere 大佬开源的 DotNetHost.Win32Metadata 库：<https://github.com/wherewhere/DotNetHost.Win32Metadata> 它做的正是这份翻译，而且是跟着 .NET SDK 版本自动生成的。本文所有的示例代码都放在 DotNetHostDemo 仓库里，文末会给出获取方式

## 三个用途

**第一个用途：在 Native AOT 下使用 .NET 非 AOT 的内容**

Native AOT 发布之后的程序是没有 JIT 的，所谓的「动态」只能靠 AOT 编译期就确定下来的东西来支撑。反射拿到的东西受限于裁剪，`Assembly.LoadFrom` 更是直接不可用。用上这个库之后，思路就变成了：AOT 的宿主只负责极速启动，所有需要动态能力的业务代码，都以普通的非 AOT 程序集形式放在旁边，由宿主在运行时把 .NET 运行时拉起来再执行

这就等于用 AOT 的启动时间换到了非 AOT 的动态功能，可以理解成一个手动版的 ReadyToRun：外层是原生代码，内层还是 JIT 的托管代码。这个用途我觉得是三个里面最有价值的

**第二个用途：在不同的 .NET 版本之间相互引用，甚至到旧 .NET Framework 版本**

.NET Framework 4.8.1 的程序是没法引用 .NET 10 的程序集的，反过来也一样。而这个库让两边可以互相把对方拉起来跑

它有一个明显缺点得先说清楚：两边的代码不能像同进程那样直接互相调用，中间隔着一层函数指针和字符串类型名，参数和返回值都要走原生能表达的形态，实际上接近跨进程通信的用法，能传的东西很有限。所以它的优点是有限的，但在一些只想复用一段新代码的场景里，它是真有用的

**第三个用途：由应用程序自己决定入口**

这个用途是我在用的时候才意识到的。既然加载哪个程序集、用哪个类型做入口、调用哪个方法，全都是宿主运行时拼出来的字符串，那宿主就可以自己定规则去选择

最典型的场景是带 OTA 自动更新的程序：只需要一个固定的启动器 exe，业务程序集按版本号分目录摆放

```
App.exe
1.0.0/App.dll
2.0.0/App.dll
```

`App.exe` 根据版本号规则决定这次动态加载哪个目录里的 `App.dll`。再配合一份配置文件，当某个版本加载失败时，直接回退到上一个可用版本

这样做有一个附带的好处：整个加载过程都在同一个入口里，统一的异常处理和崩溃上报也就有地方放了。以前做成多个 exe 互相启动的写法，是很难做到这一点的

## 库的定位

这个库已经发布成 NuGet 包了，包名就是 `DotNetHost.Win32Metadata`，本文使用的版本是 `10.0.2.1`，对应 .NET 10 版本

它在链条里的位置是这样的

```
DotNetHost.Win32Metadata  →  提供图纸
CsWin32                   →  按图纸盖房子
```

具体来说，`DotNetHost.Win32Metadata` 里装的是从 CoreCLR 原生头文件生成出来的 `.winmd` 元数据，也就是说明这份图纸。真正读图纸、把 C# 的 `delegate`、`struct`、`enum` 生成出来的，是 `Microsoft.Windows.CsWin32`。所以这两个包必须一起装，单装元数据包什么都不会发生

装完包之后，包里会自动注入一段 props，帮你把元数据和 API 文档接进 CsWin32，内容如下

```xml
<AppLocalAllowedLibraries Include="nethost" />
<ProjectionDocs Include="...\apidocs.msgpack" />
<ProjectionMetadataWinmd Include="...\DotNetHost.winmd" />
```

其中 `apidocs.msgpack` 是配套的官方注释文档，有了它，在 IDE 里悬停到 `hostfxr_initialize_for_runtime_config_fn` 这类类型上，就能看到官方文档里的说明，不用再翻头文件

## 用法就三步

### 第一步：准备一个 SDK 风格的项目

这一条是硬要求：项目文件必须是 SDK 格式，也就是带 `<Project Sdk="Microsoft.NET.Sdk">` 的那种。老式的 csproj 用不了，因为 CsWin32 的代码生成是挂在 SDK 风格项目的编译流程上的

宿主的目标框架可以随你，示例里就有一个是 .NET Framework 的

```xml
<TargetFramework>net481</TargetFramework>
```

### 第二步：装两个包

```xml
<ItemGroup>
  <PackageReference Include="DotNetHost.Win32Metadata" Version="10.0.2.1" />
  <PackageReference Include="Microsoft.Windows.CsWin32" Version="0.3.269" PrivateAssets="all" />
</ItemGroup>
```

这里再强调一次为什么是两个包。`DotNetHost.Win32Metadata` 只提供 `.winmd` 元数据文件，也就是前面说的图纸，它本身不生成任何 C# 代码；真正干活的是 `Microsoft.Windows.CsWin32`，它把图纸读进来，生成 `DllImport` 和配套的类型。少一个都跑不起来

### 第三步：写 NativeMethods.txt 声明要哪些函数

在项目根目录放一个 `NativeMethods.txt`，一行一个函数名

```
hostfxr_initialize_for_runtime_config_fn
hostfxr_get_runtime_delegate_fn
hostfxr_close_fn
load_assembly_and_get_function_pointer_fn

get_hostfxr_path
LoadLibrary
GetProcAddress
```

CsWin32 在编译时会按这个清单生成对应的 C# 声明，写多少生成多少，不写就不生成。所以这个清单要分两段看：前三行加第四行是 hostfxr 提供的函数指针类型，下面三行是要真正 P/Invoke 进去的原生函数

这里请记得：`NativeMethods.txt` 是要被当成 `AdditionalFiles` 传给 CsWin32 的。放在项目根目录的时候 CsWin32 会自己找，但 Demo 里三个宿主共用同一份清单，所以是显式加的

```xml
<AdditionalFiles Include="..\Shared\NativeMethods.txt" Link="NativeMethods.txt" />
```

到这里为止，一行 `DllImport` 都还没有手写，需要的声明已经有了

## 宿主怎么把运行时拉起来

接下来是这个方案真正的核心：把运行时拉起来并调用到托管方法。Demo 里这段逻辑抽在 `Shared/HostFxrRunner.cs` 里，三个宿主共用同一份，通过下面的写法把源码链接进来编译，而不是做成一个共享库

```xml
<Compile Include="..\Shared\HostFxrRunner.cs" Link="HostFxrRunner.cs" />
```

这个方法的签名如下。传入的六个参数里前五个都是字符串，全部由调用方给，runner 自己不做任何判断

```csharp
internal static unsafe int Run
(
    string runtimeConfigPath,
    string assemblyPath,
    string typeName,
    string methodName,
    string delegateTypeName,
    string[] args
)
```

光看这个签名就能明白这套方案的契约形态：宿主和组件之间靠「名字」对上。整个流程可以拆成五步，下面按顺序讲。具体整个 HostFxrRunner 代码，如感兴趣，可用文末提供的方法拉取我整个代码仓库获取全部代码

### 第一步：解决两个 PInvoke 类重名

先看文件一开始的引用

```csharp
using System.Runtime.Hosting.Native;
using Windows.Win32;
using Windows.Win32.Foundation;

using DotNetHost = System.Runtime.Hosting.Native.PInvoke;
using PInvoke = Windows.Win32.PInvoke;
```

这里 `get_hostfxr_path` 是由 DotNetHost.Win32Metadata 生成的，落在 `System.Runtime.Hosting.Native` 命名空间下，因为这个库把原生 API 映射到了 `System.Runtime.Hosting.Native` 这个命名空间；而 `LoadLibrary`、`GetProcAddress` 来自 CsWin32 自带的 Win32 元数据

两边都叫 `PInvoke`，所以必须起别名。示例里把项目自己的那个起名为 `DotNetHost`，把 Win32 的那个起名为 `PInvoke`。这是使用时第一个会踩到的地方，照抄这个别名写法就行

### 第二步：找到 hostfxr.dll

```csharp
const int PathMax = 260;
char* hostFxrPath = stackalloc char[PathMax];
nuint hostFxrPathSize = PathMax;
int result = DotNetHost.get_hostfxr_path(hostFxrPath, &hostFxrPathSize);
ThrowIfFailed(result, "Locating hostfxr failed.");
```

`get_hostfxr_path` 做的是「问 nethost：当前这台机器上，跟这个程序匹配的 hostfxr.dll 在哪」。生成的签名是 `char*` 加 `nuint*`，所以这里用 `stackalloc char[]` 开缓冲区，再把大小也一起传进去

这里有个细节值得留意：因为 C# 的 `char` 和 Windows 的 `wchar_t` 都是 UTF-16，生成的声明可以直接用 `stackalloc char[]` 接收文件名，不需要做任何编码转换。要是这份翻译没做好，这一步就得自己处理宽字符了

拿到路径之后先用一个缓冲区接住，而不是直接 `LoadLibrary`，是因为后面加载还需要这个路径去查导出函数

### 第三步：加载 hostfxr.dll 并取出三个函数

```csharp
using FreeLibrarySafeHandle library = new(PInvoke.LoadLibrary(new PCWSTR(hostFxrPath)), true);
if (library.IsInvalid)
{
    throw new InvalidOperationException("Loading hostfxr failed.");
}

hostfxr_initialize_for_runtime_config_fn initialize = GetExport<hostfxr_initialize_for_runtime_config_fn>(library, "hostfxr_initialize_for_runtime_config");
hostfxr_get_runtime_delegate_fn getRuntimeDelegate = GetExport<hostfxr_get_runtime_delegate_fn>(library, "hostfxr_get_runtime_delegate");
hostfxr_close_fn close = GetExport<hostfxr_close_fn>(library, "hostfxr_close");
```

`LoadLibrary` 返回的不是裸句柄，而是 `FreeLibrarySafeHandle`。这是 CsWin32 生成代码时顺手带来的好处：原生句柄被包装成 `SafeHandle`，用 `using` 包一层，出作用域时会自动 `FreeLibrary`，不用担心异常路径下漏掉释放

取导出的辅助方法也一并看一眼

```csharp
private static TDelegate GetExport<TDelegate>(FreeLibrarySafeHandle library, string name) where TDelegate : Delegate
{
    FARPROC address = PInvoke.GetProcAddress(library, name);
    if (address.IsNull)
    {
        throw new MissingMethodException($"The hostfxr export '{name}' was not found.");
    }

    return address.CreateDelegate<TDelegate>();
}
```

`GetProcAddress` 拿到的是 `FARPROC`，直接 `CreateDelegate<TDelegate>()` 就变成了强类型的委托，泛型参数就是 `NativeMethods.txt` 里声明过的那些 `hostfxr_xxx_fn` 类型。这一步之所以能这么干净，是因为元数据里那些函数指针类型已经被生成成了 C# 的 `delegate`，否则这里得手写 `Marshal.GetDelegateForFunctionPointer` 再转 `IntPtr`

### 第四步：初始化运行时，拿到加载程序集的委托

```csharp
hostfxr_handle context = default;
fixed (char* runtimeConfigPathPointer = runtimeConfigPath)
{
    result = initialize(runtimeConfigPathPointer, null, &context);
}

ThrowIfFailed(result, "Initializing the .NET runtime failed.");
if (context.IsNull)
{
    throw new InvalidOperationException("hostfxr returned an empty runtime context.");
}
```

`initialize` 接收的就是那个 `runtimeconfig.json` 路径，这份配置文件决定了要启动的运行时是哪个版本，我在后一节的部署部分会单独讲

初始化成功后会返回一个 `context`，类型是 `hostfxr_handle`。这个句柄是后面所有操作的入口，也是最后要关掉的东西，所以整个加载过程都写在 `try` 里面，`finally` 里负责关闭

拿到 context 之后，向运行时索取「加载程序集并取函数指针」的委托

```csharp
FARPROC loadAssemblyPointer = default;
result = getRuntimeDelegate
(
    context,
    hostfxr_delegate_type.hdt_load_assembly_and_get_function_pointer,
    &loadAssemblyPointer
);
ThrowIfFailed(result, "Getting the assembly loader failed.");

load_assembly_and_get_function_pointer_fn loadAssembly = loadAssemblyPointer.CreateDelegate<load_assembly_and_get_function_pointer_fn>();
```

为什么中间要绕一下 `FARPROC`？因为 `hostfxr_get_runtime_delegate` 返回的是一块裸的委托指针，同一份 API 可以按 `hostfxr_delegate_type` 返回好几种不同的委托，要转成哪种由调用方自己决定

### 第五步：加载托管入口并调用

```csharp
FARPROC entryPointPointer = default;
fixed (char* assemblyPathPointer = assemblyPath)
fixed (char* typeNamePointer = typeName)
fixed (char* methodNamePointer = methodName)
fixed (char* delegateTypeNamePointer = delegateTypeName)
{
    result = loadAssembly
    (
        assemblyPathPointer,
        typeNamePointer,
        methodNamePointer,
        delegateTypeNamePointer,
        null,
        &entryPointPointer
    );
}

ThrowIfFailed(result, "Loading the managed entry point failed.");
ComponentEntryPoint entryPoint = entryPointPointer.CreateDelegate<ComponentEntryPoint>();
return entryPoint(args);
```

`loadAssembly` 的六个参数是全篇最需要理解的地方，因为它们把「同一个进程里两个运行时如何握手」这件事说清楚了

- 程序集路径：要加载哪个 dll

- 类型名：入口类型，格式是 `类型全名, 程序集名`

- 方法名：入口方法名

- 委托类型名：托管侧声明的委托类型全名，用来约束方法签名

- 配置参数：这里是 `null`

- 出参：拿到的函数指针

这五样东西里有四样是字符串。也就是说，两个运行时之间唯一的契约是「名字」，这也是前面说它「接近跨进程」的原因：类型安全完全靠这套名字对得上，对不上就会失败

这里还有个小坑请记得：官方的示例里用的是 `Unsafe.AsPointer(ref MemoryMarshal.GetReference(span))` 这种写法来传字符串，原因是它手上是一个 `ReadOnlySpan`，而 `ReadOnlySpan` 是没法 `fixed` 的。Demo 里把参数换成了普通的 `string`，就可以直接用 `fixed` 了，两种写法按自己的数据类型挑就行

最后一行的 `ComponentEntryPoint` 是宿主和组件约定好的委托签名，它整个定义就一行

```csharp
internal delegate int ComponentEntryPoint(string[] args);
```

宿主和组件两边都要有这个签名完全一致的委托类型，这算是整套方案里唯一的「手工契约」

### 出错时只能看十六进制错误码

```csharp
private static void ThrowIfFailed(int result, string message)
{
    if (result != 0)
    {
        throw new InvalidOperationException($"{message} hostfxr result: 0x{result:X8}.");
    }
}
```

hostfxr 的错误码都是 `0x80008xxx` 这一族 HRESULT，直接抛 `int` 看不出任何信息，所以这里统一按 `X8` 十六进制格式化，查错时再去对照 hostfxr 的官方错误码表。官方示例这一步用的是 `Marshal.ThrowExceptionForHR`，那样能拿到 HRESULT 的文本描述，但拿不到「是哪一步挂的」，这里改成带上上下文文案的写法，至少知道是哪一步失败了

## 什么是组件

前面一直说「由宿主加载组件」，这里需要把组件这个概念说清楚

在这个方案里，**组件指的是被宿主加载的那个普通的 .NET 10 类库程序集**，也就是宿主启动的 .NET 10 运行时里真正干活的 dll。它不是 NuGet 包，也不用做成特殊的项目类型，就是一个普通的 `Microsoft.NET.Sdk` 类库

这里补一句：如果加载的 .NET 10 是一个带 exe 入口的行不行？那肯定也是行的，而且不需要 `GenerateRuntimeConfigurationFiles` 的存在，因为 exe 本来就生成自己的 runtimeconfig.json，直接的 Main 也就是导出了，方法名写 `<Main>$` 就能取到。官方示例 `DotNetHost.Win32Metadata.Test` 就是这么干的，自己加载自己的 exe，类型写成 `Program, DotNetHost.Win32Metadata.Test`，方法名写成 `<Main>$`。只是说正常做被加载的，很少是带 exe 入口的，而是做组件而已

看一下 Alpha 组件的全部内容

```csharp
namespace ManagedComponent.Alpha;

public delegate int ComponentEntryPoint(string[] args);

public static class Component
{
    public static int Run(string[] args)
    {
        Console.WriteLine($"Alpha component is running on .NET {Environment.Version}.");
        Console.WriteLine($"Arguments: {string.Join(", ", args)}");
        return 101;
    }
}
```

一个委托、一个静态类、一个静态方法，就这么多。它对自己的处境一无所知，也不需要知道宿主是什么

组件这边的项目文件只需要在普通类库的基础上加一句

```xml
<GenerateRuntimeConfigurationFiles>true</GenerateRuntimeConfigurationFiles>
```

这一句是为了让组件生成自己的 `runtimeconfig.json`，也就是前面 hostfxr 初始化时要用的那份配置。Beta 组件和 Alpha 完全一样，只是命名空间和返回值不同

回到最开始那个「一个进程里两个运行时」的观察，现在可以把它讲完整了。组件里打印的 `Environment.Version` 是 `10.x`。而在 .NET Framework 宿主这一侧，如果加一句打印自己的版本

```csharp
Console.WriteLine($"FrameworkHost is running on .NET {Environment.Version}.");
```

打印出来是 `4.0.30319`。同一个进程里，两个不同的运行时各自报着不同的版本号，这正是这套方案最有意思的地方，也是前面三个用途能成立的根本原因

### 路由版宿主是怎么决定入口的

前面说的第三个用途「应用程序决定入口」，在 Demo 里对应的是 `NativeAotRouter`。它的做法是先用参数选出一个描述对象，再用描述对象去拼所有需要的符号

描述对象只装三样东西，全是字符串

```csharp
internal sealed record ComponentDescriptor(string AssemblyName, string TypeName, string DelegateTypeName);
```

选的过程就是一次 `switch`，如下面代码。这里我给每个组件把程序集名、入口类型名、委托类型名都写在了一起，原因看下一步就明白了

```csharp
ComponentDescriptor? component = args[0].ToLowerInvariant() switch
{
    "alpha" => new
    (
        "ManagedComponent.Alpha",
        "ManagedComponent.Alpha.Component",
        "ManagedComponent.Alpha.ComponentEntryPoint"
     ),
    "beta" => new
    (
        "ManagedComponent.Beta",
        "ManagedComponent.Beta.Component",
        "ManagedComponent.Beta.ComponentEntryPoint"
      ),
    _ => null
};
```

选中之后，路径和完整的类型名都是从这三个字段拼出来的

```csharp
return HostFxrRunner.Run
(
    Path.Combine(baseDirectory, $"{component.AssemblyName}.runtimeconfig.json"),
    Path.Combine(baseDirectory, $"{component.AssemblyName}.dll"),
    $"{component.TypeName}, {component.AssemblyName}",
    "Run",
    $"{component.DelegateTypeName}, {component.AssemblyName}",
    componentArguments
);
```

这段代码本身很朴素，但它说明了真正重要的一点：因为入口的全部信息都是运行时可拼的字符串，所以「用哪个程序集」这个决定可以留给程序自己在运行时做。前面 OTA 多版本回退的场景，无非就是把 `switch` 换成读配置文件和版本号比较，把失败的情况接个 `catch` 再换下一个版本重试

## 宿主项目文件里的约定

库的用法只有三步，但真正让人卡住的往往是部署，所以这部分单独讲

### 必须自己带上 nethost.dll

`get_hostfxr_path` 这个函数住在 `nethost.dll` 里，而这个 dll 不在 NuGet 包里，得自己从 .NET SDK 里取出来放到输出目录

```xml
<PropertyGroup>
  <AppHostPath>$(NetCoreTargetingPackRoot)\Microsoft.NETCore.App.Host.win-x64\$(BundledNETCoreAppPackageVersion)\runtimes\win-x64\native</AppHostPath>
</PropertyGroup>

<ItemGroup>
  <ReferenceCopyLocalPaths Include="$(AppHostPath)\nethost.dll" />
</ItemGroup>
```

`hostfxr.dll` 不用自己操心，nethost 会帮你找到

这里要提醒的是位数问题：`nethost.dll` 是分 win-x64 和 win-x86 的，两者不能混用。如果宿主进程是 32 位的，却把 64 位的 `nethost.dll` 拷到旁边，就会在 `LoadLibrary` 那一步直接失败，而且错误信息只会告诉你加载不了，不会告诉你是位数不对。.NET Framework 默认是 AnyCPU，在 64 位系统上会跑成 64 位进程，很容易和 x86 的 dll 混在一起

所以 Demo 在 `Directory.Build.props` 里把所有项目统一钉成了 x64

```xml
<Platforms>x64</Platforms>
<PlatformTarget>x64</PlatformTarget>
```

上面那段 `AppHostPath` 里的 `win-x64` 也是同一个意思，必须和宿主的位数一致

### 别忘了 runtimeconfig.json

前面说过，hostfxr 初始化时必须有一份 `runtimeconfig.json` 告诉它要启动哪个运行时

```json
{
  "runtimeOptions": 
  {
    "tfm": "net10.0",
    "framework":
     {
      "name": "Microsoft.NETCore.App",
      "version": "10.0.0"
    }
  }
}
```

这份文件由组件项目在编译时生成，但它生成在组件的输出目录里，宿主运行时读的是宿主自己旁边的路径，所以要把它拷过去

### 把组件连同配置一起拷到宿主输出目录

这部分没有现成机制，得自己写 MSBuild Target。宿主先确保组件被编译

```xml
<Target Name="BuildManagedComponent" BeforeTargets="Build">
  <MSBuild Projects="..\ManagedComponent.Alpha\ManagedComponent.Alpha.csproj" Targets="Build" Properties="Configuration=$(Configuration)" />
</Target>
```

再把组件产物拷过去

```xml
<Target Name="CopyManagedComponent" AfterTargets="Build">
  <ItemGroup>
    <ManagedComponentFiles Include="..\ManagedComponent.Alpha\bin\$(Configuration)\net10.0\ManagedComponent.Alpha.dll" />
    <ManagedComponentFiles Include="..\ManagedComponent.Alpha\bin\$(Configuration)\net10.0\ManagedComponent.Alpha.runtimeconfig.json" />
  </ItemGroup>
  <Copy SourceFiles="@(ManagedComponentFiles)" DestinationFolder="$(OutputPath)" SkipUnchangedFiles="true" />
</Target>
```

dll 和 runtimeconfig.json 要成对拷贝，少一份就在运行时挂掉。另外 Native AOT 的宿主还多一个坑：AOT 的产物在 `publish` 目录而不是 `bin` 目录，所以 `CopyManagedComponent` 要同时挂到 `Build` 和 `Publish` 两个时机上，并且分别往 `$(OutputPath)` 和 `$(PublishDir)` 拷

```xml
<Target Name="CopyManagedComponent" AfterTargets="Build;Publish">
  <ItemGroup>
    <ManagedComponentFiles Include="..\ManagedComponent.Alpha\bin\$(Configuration)\net10.0\ManagedComponent.Alpha.dll" />
    <ManagedComponentFiles Include="..\ManagedComponent.Alpha\bin\$(Configuration)\net10.0\ManagedComponent.Alpha.runtimeconfig.json" />
  </ItemGroup>
  <Copy SourceFiles="@(ManagedComponentFiles)" DestinationFolder="$(OutputPath)" SkipUnchangedFiles="true" />
  <Copy SourceFiles="@(ManagedComponentFiles)" DestinationFolder="$(PublishDir)" SkipUnchangedFiles="true" Condition="'$(PublishDir)' != ''" />
</Target>
```

这个 `Condition` 不能省，因为 `dotnet build` 的时候 `$(PublishDir)` 是空的，不判断条件它会把文件拷到一个不存在的路径上去

## 实际用起来的感觉

### 顺手的地方

- **真的不用碰 C++**。整个 Demo 里没有一个 C++ 项目，也没有一行手写的 `DllImport`，需要的声明全在 `NativeMethods.txt` 里列个名字就行

- **生成的类型是强类型的**。`hostfxr_handle`、`hostfxr_delegate_type` 这些在 C# 里都是正常的类型，`CreateDelegate<T>` 直接给出强类型委托，不用到处跟 `IntPtr` 打交道

- **句柄被包装成 SafeHandle**。`LoadLibrary` 返回 `FreeLibrarySafeHandle`，`using` 一包就完事，这是手抄版最容易漏的东西

- **跟着 SDK 版本走**。元数据的版本号是绑在 `$(BundledNETCoreAppPackageVersion)` 上的，SDK 升级之后包也跟着发新版本，不需要自己维护一份翻译稿

- **有注释**。包里带了 `apidocs.msgpack`，悬停能看到官方文档原文，查 API 语义不用翻头文件

- **宿主端的目标框架很自由**。这一点在 .NET Framework 4.8.1 上体现得最明显，因为这个库跟 .NET Framework 完全独立，只要项目是 SDK 风格就能用

### 不太舒服的地方

- **两个 `PInvoke` 类重名**。每次新建宿主项目都要重新想一遍别名，虽然只是抄一遍 `using`，但第一次遇到时确实会愣一下

- **nethost.dll 的位数要对上**。库能帮你生成代码，但没法帮你判断进程位数，错了就只能在 `LoadLibrary` 看到一句没有信息量的失败

- **部署要自己写 MSBuild**。组件 dll 和它的 runtimeconfig.json 都要拷到宿主输出目录，AOT 宿主还要多处理一层 `PublishDir`，这几行 Target 基本是每个项目都要复制一遍

- **运行时拉起来就关不掉**。`hostfxr_close` 关掉的只是那个 context，已经加载进进程的 .NET 运行时和程序集不会卸载。所以这个方案适合「启动时决定一次入口」，不适合反复加载卸载。前面说的 OTA 回退场景，回退也只能发生在还没成功初始化的阶段，一旦某个版本跑起来了，想换版本基本就得重启进程

- **出错时不好定位**。拿到手的只有 `0x80008xxx` 这样的错误码，得去翻 hostfxr 的错误码表。好在示例在每次失败时都带了上下文文案，至少知道是哪一步挂的

- **互调用近似于跨进程**。没办法像直接调用方法或属性的方式直接访问，只能按照跨进程协约方式工作。这就限制了很多实现情景了。不会比多进程调用的方案有非常明显的提升

总的来说，`DotNetHost.Win32Metadata` 真正的价值是让不同形态的 .NET 能互相加载：Native AOT 与非 AOT 之间、新版本与旧版本之间、.NET 10 与 .NET Framework 4.8.1 之间

## 获取代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0ade828087bb0f410b0bdeebbbaeed4ab5c6da62/Workbench/DotNetHostDemo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0ade828087bb0f410b0bdeebbbaeed4ab5c6da62/Workbench/DotNetHostDemo) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0ade828087bb0f410b0bdeebbbaeed4ab5c6da62
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0ade828087bb0f410b0bdeebbbaeed4ab5c6da62
```

获取代码之后，进入 Workbench/DotNetHostDemo 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html)




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。