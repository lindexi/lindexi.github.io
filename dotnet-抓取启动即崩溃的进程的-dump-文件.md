
# dotnet 抓取启动即崩溃的进程的 dump 文件

本文介绍我开源的 ProcdumpExec 工具，用来解决目标进程启动即崩溃、ProcDump 还来不及附加就已经退出，从而抓不到 dump 文件的问题

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

本文内容由人类主导 AI 辅助编写

## 背景

在使用 ProcDump 抓 dump 的时候，最常用的做法是 `procdump -e -ma <PID>`，也就是把 ProcDump 附加到一个已经在运行的进程上。可是对于那种一启动就崩溃的进程，这个做法就无能为力了。原因是此时我们还没有拿到进程的 PID，等我们想办法查到 PID 再手动附加，进程早就因为访问冲突之类的异常退出了

有伙伴会说，ProcDump 本身不是提供了 `-e -w` 参数用来等待新进程创建吗？确实如此，这个参数组合的意思是，让 ProcDump 守在那里，等有目标进程被创建时再附加进去。看似可以解决问题，但这里有一个很隐蔽的坑。从目标进程被创建，到 ProcDump 真正完成调试器附加，中间是需要一小段时间的。在这段时间里，目标进程的主线程可是正常在跑的，如果崩溃逻辑发生在进程启动的最开头，那这段窗口期就足够它死给你看了

另一个坑是一些多进程的程序，也很难通过 -w 等待启动的方式进行附加

换句话说，光靠 ProcDump 自己，是没有办法保证「进程一启动就被监控」的

那要怎么办呢？关键在于 Windows 提供了一个 `CREATE_SUSPENDED` 标志，允许我们创建一个主线程处于挂起状态的进程。挂起状态下，目标进程不会执行任何用户代码。于是我们就有足够的时间，先把 ProcDump 附加进去，确认附加完成之后，再把目标线程恢复。这样一来，目标进程执行的第一条用户指令，就已经处在 ProcDump 的监控之下了

本文要介绍的 ProcdumpExec，做的就是这套编排工作

## 整体思路

整个工具的流程可以拆成下面几步：

1. 解析命令行，用 `--` 把 ProcDump 参数和目标程序命令行分开
2. 用 `CreateProcessW` 加 `CREATE_SUSPENDED` 把目标程序挂起创建出来
3. 根据自身架构，选择同目录下的 `procdump64.exe` 或 `procdump.exe`
4. 把 ProcDump 参数原样传过去，末尾补上新创建进程的 PID
5. 启动 ProcDump，然后轮询等待它完成附加
6. 附加完成后，调用 `ResumeThread` 恢复目标进程的主线程
7. 等待 ProcDump 退出，并把它的退出码原样返回

顺序是很有讲究的。第 2 步先把进程挂起，才能保证第 5 步附加之前目标进程没有机会跑起来；第 6 步恢复线程一定要放在第 5 步之后，否则就又回到「进程跑在监控前」的老问题上了

下面咱来逐个拆开看实现细节

## 拆分命令行

工具的第一个动作，是从命令行里找到 `--` 分隔符。`--` 之前的参数会原样透传给 ProcDump，之后的第一个参数是目标程序，再后面的参数则给目标程序

```csharp
var separatorIndex = Array.IndexOf(args, "--");
if (separatorIndex < 0 || separatorIndex == args.Length - 1)
{
    Console.Error.WriteLine("Usage: ProcdumpExec [ProcDump arguments] -- <executable> [arguments]");
    return 1;
}
```

这里同时处理了两种非法输入：完全没有 `--`，以及 `--` 出现在最后一个位置导致后面没有目标程序。这两种情况都直接打印用法并返回 `1`，避免后续逻辑拿到非法数据。之所以要引入 `--` 这么一个约定，是因为 ProcDump 自己本身就有一大堆参数，如果不用分隔符，根本没法可靠地区分「哪个参数是给 ProcDump 的」和「哪个参数是给目标程序的」

找到分隔位置之后，`--` 之后的参数被拿去构造目标程序的命令行：

```csharp
var targetCommandLine = BuildCommandLine(args.AsSpan(separatorIndex + 1));
```

## 挂起创建目标进程

接下来是最核心的一步，用 `CreateProcessW` 创建目标进程，并且带上 `CREATE_SUSPENDED` 标志：

```csharp
const uint CreateSuspended = 0x00000004;
```

```csharp
var startupInfo = new StartupInfo
{
    Size = (uint)Marshal.SizeOf<StartupInfo>()
};

if (!CreateProcess
    (
        applicationName: null,
        commandLine: targetCommandLine,
        processAttributes: 0,
        threadAttributes: 0,
        inheritHandles: true,
        creationFlags: CreateSuspended,
        environment: 0,
        currentDirectory: null,
        startupInfo,
        out var processInformation
    ))
{
    throw new Win32Exception(Marshal.GetLastWin32Error());
}
```

这里有几点值得说明

第一是 `StartupInfo` 的 `Size` 字段。这个结构体是 Win32 API 里很经典的一个「自描述尺寸」结构，调用方必须自己把结构体大小填进去，操作系统才知道该按多大来读取。如果这个值填错，`CreateProcessW` 会直接失败，而且错误码往往不够直观，排查起来比较费劲

第二是 `applicationName` 传了 `null`，把要启动的程序完全交给 `commandLine` 来表达。这也是 Win32 上比较惯用的做法，因为程序路径本身也可能包含空格，统一交给命令行去解析更省心

第三是 `inheritHandles: true`，让目标进程继承当前控制台的句柄。这样 ProcDump 和目标程序就能直接往当前控制台输出，ProcdumpExec 自己不需要做任何标准输出的重定向或转发，子进程写什么，使用者就看到什么

第四是 `CreateSuspended` 这个 `0x00000004` 的标志位。它对应的就是 `CREATE_SUSPENDED`，让新进程的主线程创建后处于挂起状态，不执行任何代码

一旦创建失败，就直接抛出 `Win32Exception`，把 `Marshal.GetLastWin32Error()` 拿到的错误码带上，方便定位是权限问题还是程序路径问题

## 启动 ProcDump 并传入 PID

目标进程挂起在那里之后，就要请 ProcDump 出场了。首先要确定用哪个 ProcDump：

```csharp
var procDumpFileName = Environment.Is64BitProcess ? "procdump64.exe" : "procdump.exe";
var procDumpPath = Path.Combine(AppContext.BaseDirectory, procDumpFileName);
```

ProcDump 有两个版本，64 位的 `procdump64.exe` 和 32 位的 `procdump.exe`。这里用 `Environment.Is64BitProcess` 来判断当前 ProcdumpExec 进程自身的位数，进而选择对应的 ProcDump。这样映射是成立的，因为一个 x86 版本的 ProcdumpExec 即便跑在 64 位系统上，它自身也是 32 位进程，`Is64BitProcess` 会返回 `false`，于是它会去拿 32 位的 `procdump.exe`，和目标程序的位数保持一致

路径则是用 `AppContext.BaseDirectory` 拼出来的，也就是说 ProcDump 必须和 ProcdumpExec 放在同一个目录里。这一点在发布的时候需要特别注意，两种位数的文件布局是这样的：

```text
release_win-x64\
    ProcdumpExec64.exe
    procdump64.exe

release_win-x86\
    ProcdumpExec.exe
    procdump.exe
```

接着就是把参数拼装起来：

```csharp
var startInfo = new ProcessStartInfo
{
    FileName = procDumpPath,
    UseShellExecute = false
};

for (var i = 0; i < separatorIndex; i++)
{
    startInfo.ArgumentList.Add(args[i]);
}

startInfo.ArgumentList.Add(processInformation.ProcessId.ToString(CultureInfo.InvariantCulture));
```

`--` 之前的参数被逐个加到参数列表里，保持原样。然后关键的一行，是把新创建进程的 PID 追加到末尾。这正是 ProcDump 期望的调用形式，等价于是我们在手动敲 `procdump64.exe -accepteula -ma -e 1 <目标进程 PID>`

这里有两个小细节。一是用 `ArgumentList` 而不是自己拼一个字符串，因为 `ArgumentList` 会由 .NET 负责转义每一个参数，我们不用操心空格和引号的问题。二是 PID 转字符串的时候显式指定了 `CultureInfo.InvariantCulture`，避免在某些地区设置下出现千位分隔符之类的意外

## 等待 ProcDump 完成附加

ProcDump 进程启动之后，还不能立刻恢复目标进程，得先确认它已经附加上了。判断的依据就是目标进程身上有没有调试器：

```csharp
static void WaitForProcDumpToAttach(Process procDump, nint targetProcessHandle)
{
    while (!procDump.HasExited)
    {
        if (!CheckRemoteDebuggerPresent(targetProcessHandle, out var isDebuggerPresent))
        {
            throw new Win32Exception(Marshal.GetLastWin32Error());
        }

        if (isDebuggerPresent)
        {
            return;
        }

        Thread.Sleep(10);
    }
}
```

`CheckRemoteDebuggerPresent` 会检查指定进程当前是否正被调试。ProcDump 抓 dump 的方式本质上就是把自己作为调试器附加到目标进程上，所以在它完成附加的那一刻，这个函数就会返回 `true`

循环里每次等 10 毫秒再重试，这个间隔既不至于太占 CPU，又能保证附加完成后的响应足够快。循环条件 `!procDump.HasExited` 则是一个兜底：万一 ProcDump 因为参数写错等原因提前退出了，循环会自然结束，不至于死等下去

## 恢复目标进程

确认附加完成之后，就可以放行目标进程了：

```csharp
if (ResumeThread(processInformation.ThreadHandle) == uint.MaxValue)
{
    throw new Win32Exception(Marshal.GetLastWin32Error());
}

resumed = true;
procDump.WaitForExit();
return procDump.ExitCode;
```

`ResumeThread` 返回 `uint.MaxValue`（也就是 `-1`）表示调用失败，此时同样抛异常把错误码带出来。恢复成功之后，目标进程的主线程才开始执行它真正的代码，而这时候 ProcDump 已经在监听它了，于是「启动即崩溃」的那一下也能被完整地捕获下来

`resumed` 这个标记很重要，它记录了目标线程到底有没有被恢复过。记录它的原因是，后面的清理逻辑要依赖这个状态

最后调用 `procDump.WaitForExit()` 完整地等待 ProcDump 退出，再把 ProcDump 的退出码原样返回。这样 ProcdumpExec 就能在脚本里当成 ProcDump 的包装来使用，退出码的语义保持一致

## 异常路径的清理

整个主体逻辑被包在一个 `try` 里，配套的 `finally` 负责收尾：

```csharp
finally
{
    if (!resumed)
    {
        TerminateProcess(processInformation.ProcessHandle, 1);
    }

    CloseHandle(processInformation.ThreadHandle);
    CloseHandle(processInformation.ProcessHandle);
}
```

如果流程中途抛了异常，导致目标线程一直没被恢复，那目标进程就会以一个挂起的状态永久卡在那里，既不会运行也不会退出，成为一个很难被发现的问题。所以在 `finally` 里检查 `resumed`，只要没恢复过，就调用 `TerminateProcess` 把这个挂起的进程杀掉

另外，`processInformation` 里拿到的进程句柄和线程句柄都是内核对象，用完必须调用 `CloseHandle` 释放，否则就会造成句柄泄漏。这一步无论成功还是异常都要执行，所以放在 `finally` 里最合适

## 拼接命令行的转义规则

目标程序的命令行不是简单地把参数用空格连起来就完事的，因为 Windows 的命令行解析有一套自己的转义规则。`BuildCommandLine` 负责把参数数组拼成一个字符串

```csharp
static StringBuilder BuildCommandLine(ReadOnlySpan<string> arguments)
{
    var commandLine = new StringBuilder();

    foreach (var argument in arguments)
    {
        if (commandLine.Length > 0)
        {
            commandLine.Append(' ');
        }

        AppendQuotedArgument(commandLine, argument);
    }

    return commandLine;
}
```

除了第一个参数之外，每个参数前面补一个空格，然后把参数本身交给 `AppendQuotedArgument` 处理。真正有讲究的是这个方法

```csharp
if (argument.Length > 0 && argument.IndexOfAny([' ', '\t', '"']) < 0)
{
    commandLine.Append(argument);
    return;
}
```

先做一个快速判断：如果一个参数非空，并且里面既没有空格、制表符，也没有双引号，那它不需要任何转义，直接原样追加即可。这是最常见的情况，把这条快路径单独拎出来，既简单又高效

对于需要转义的参数，就要走完整的流程了

```csharp
commandLine.Append('"');
var backslashCount = 0;

foreach (var character in argument)
{
    if (character == '\\')
    {
        backslashCount++;
        continue;
    }

    if (character == '"')
    {
        commandLine.Append('\\', backslashCount * 2 + 1);
        commandLine.Append('"');
        backslashCount = 0;
        continue;
    }

    commandLine.Append('\\', backslashCount);
    commandLine.Append(character);
    backslashCount = 0;
}

commandLine.Append('\\', backslashCount * 2);
commandLine.Append('"');
```

这段逻辑遵循的是 Windows 上 `CommandLineToArgvW` 的解析规则，核心是反斜杠和双引号的配合关系。规则可以拆成三条来理解

第一条，遇到普通字符时，把它前面攒着的反斜杠原样输出，再输出这个字符。反斜杠只有在双引号前面才有特殊含义，跟在普通字符前面时就是一个普通的反斜杠，所以攒了几个就补回几个

第二条，遇到双引号时，前面的反斜杠数量需要变成原来的两倍再加一。翻倍是为了让解析方能认出「这些反斜杠是字面量」，加的那一个则是用来转义这个双引号本身的。这就是 `backslashCount * 2 + 1` 的来历

第三条，参数结束、补上结尾引号之前，要把剩下的反斜杠数量翻倍，对应 `backslashCount * 2`。这是因为结尾的引号是真正的边界引号，不需要转义，但紧挨着它的反斜杠会被解析方当成转义序列看待，所以必须翻倍，才能让原本位于参数末尾的反斜杠正确地保留下来

用这套规则处理下来，像空参数、包含空格的路径、参数内部带引号、以及以反斜杠结尾的路径这些情况，都能正确还原

## 使用方式

工具的用法非常简单：

```text
ProcdumpExec64.exe [ProcDump 参数] -- <目标程序> [目标程序参数]
ProcdumpExec.exe   [ProcDump 参数] -- <目标程序> [目标程序参数]
```

比如要捕获一个启动即崩溃的进程，可以这样写：

```cmd
ProcdumpExec64.exe -accepteula -ma -e 1 -- Xxxx.exe args1 args2
```

它完全等价于先拿到目标进程 PID，再手动执行：

```text
procdump64.exe -accepteula -ma -e 1 <目标进程 PID>
```

区别只是 PID 的获取和附加的时机，都由 ProcdumpExec 帮你编排好了

工具还会完整等待 ProcDump 退出，并且原样返回它的退出码。ProcDump 和目标程序都继承当前控制台，ProcdumpExec 自己不重定向也不修改它们的标准输出和标准错误，所以你在控制台里看到的输出，就是 ProcDump 原本会打印的内容

## 构建与发布

如果想自己构建，这是一个 .NET NativeAOT 的项目，发布命令如下

发布 x64 版本：

```cmd
dotnet publish Code\ProcdumpExec\ProcdumpExec.csproj -c Release -r win-x64
```

产物在 `artifacts\publish\ProcdumpExec\release_win-x64\ProcdumpExec64.exe`，记得把 `procdump64.exe` 放到它旁边

发布 x86 版本：

```cmd
dotnet publish Code\ProcdumpExec\ProcdumpExec.csproj -c Release -r win-x86
```

产物在 `artifacts\publish\ProcdumpExec\release_win-x86\ProcdumpExec.exe`，同样需要把 `procdump.exe` 放在一起

x64 和 x86 两个版本的程序集名字也不一样，这是通过项目文件里的条件来控制的：

```xml
<AssemblyName Condition="'$(RuntimeIdentifier)' == 'win-x64'">ProcdumpExec64</AssemblyName>
<AssemblyName Condition="'$(RuntimeIdentifier)' != 'win-x64'">ProcdumpExec</AssemblyName>
```

这样构建产物的名字就能跟着目标平台区分开，使用者一眼就能看出该用哪个

## 测试程序

仓库里带了两个测试程序，方便验证工具是否正常工作。其中 `ProcdumpExec.TestTarget` 用来模拟各种被捕获的目标进程，它的逻辑很精简

```csharp
if (args is ["crash"])
{
    Marshal.WriteInt32(0, 42);
}
```

传入 `crash` 参数时，它会直接往空指针地址写数据，立刻触发访问冲突。这正是「启动即崩溃」的场景，用来验证 ProcdumpExec 是不是真的能在崩溃发生前就把 ProcDump 附加好

它还能按指定退出码退出，以及把收到的参数打印出来：

```csharp
if (args is ["exit", var exitCode])
{
    return int.Parse(exitCode);
}

Console.WriteLine(string.Join('|', args));
return 0;
```

后面的参数回显可以用来验证命令行的转义是否正确，因为 `Main` 拿到的 `args` 已经是被系统解析过的结果，如果转义写错了，这里回显出来的参数就会和预期对不上

另一个测试程序 `ProcdumpExec.TestRunner` 则是一个简单的启动器，负责启动另一个程序并把它自己的退出码打印出来：

```csharp
using var process = Process.Start(startInfo)!;
process.WaitForExit();
Console.WriteLine($"PROCDUMP_EXEC_EXIT_CODE={process.ExitCode}");
return 0;
```

它的作用是把 ProcdumpExec 返回的退出码单独抓出来打印，方便在自动化测试里断言退出码的传递是否符合预期

比如用测试程序立即触发访问冲突，并让 dump 写在当前目录：

```cmd
ProcdumpExec64.exe -accepteula -ma -e 1 -- ProcdumpExec.TestTarget.exe crash
```

成功的话可以看到类似这样的输出：

```text
Exception: C0000005.ACCESS_VIOLATION
Dump 1 initiated: C:\Temp\ProcdumpExecDumps\ProcdumpExec.TestTarget.exe_....dmp
Dump 1 complete
```

到这里，一个启动即崩溃的进程就被完整地捕获下来了。如果你也有抓不到 dump 的烦恼，不妨试试这个工具，或者把它当成一个理解 Windows 进程创建和调试附加的小例子

本文代码放在 github 上的 UWP 仓库的 master 分支，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin git@github.com:lindexi/UWP.git
git pull origin master
```

获取代码之后，进入 app\Tool\ProcdumpExec\ 文件夹，即可获取到源代码

如果不想自己构建，也可以直接到 <https://github.com/lindexi/UWP/releases/tag/ProcdumpExec_1.0.0> 下载我构建好的压缩包使用

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。