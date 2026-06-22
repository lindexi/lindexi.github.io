---
title: dotnet WPF 实现端口占用检测与自动查找可用端口
description: 本文介绍一个基于 .NET WPF 的端口检测器工具，讲解其核心的端口检测原理、如何通过 Windows API 查询端口占用进程，以及自动查找可用端口的实现方法。
tags: WPF,dotnet
category: 
---

<!-- 发布 -->
<!-- 博客 -->

本文内容由人类主导 AI 辅助编写

## 背景

在开发过程中，经常需要确认某个端口是否被占用——启动 HTTP 服务前要看看 8080 有没有空闲，配置监听地址时要确认 127.0.0.1 和 0.0.0.0 两种绑定方式各自的状态。

传统的做法是打开命令行敲 `netstat -ano | findstr :端口号`，然后去任务管理器找 PID。但这种方法有几个盲区：如果端口被系统保留（比如被 Hyper-V 或 WSL 的排除端口范围覆盖），或者被 HTTP.sys 等内核组件占用，netstat 的输出可能看不到任何记录，却依然无法绑定。

于是就有了这个端口检测器工具。它通过真正去尝试监听端口来判断占用情况，并结合 Windows API 查询占用进程，还能一键自动查找当前可用的端口。

## 最核心的检测原理：直接去 Bind

检测端口是否空闲，最直接的方法不是去查系统表，而是自己动手去绑一下。能绑上去就是空闲，绑不上去就是被占用。比起分析 netstat 的输出、查注册表、查系统事件日志，直接 Bind 一把是最可靠的——因为你真正模拟了应用程序绑定的行为，不会有任何误判。

`TryListen` 方法的完整代码如下，可以先整体看一眼：

```csharp
private static ProbeResult TryListen(PortProbe probe, int port)
{
    try
    {
        using var socket = new Socket(AddressFamily.InterNetwork, probe.SocketType, probe.ProtocolType)
        {
            ExclusiveAddressUse = true
        };

        socket.Bind(new IPEndPoint(probe.Address, port));

        if (probe.NeedsListen)
        {
            socket.Listen(1);
        }

        return new ProbeResult(probe.Protocol, probe.Address, false, "监听成功。", null);
    }
    catch (SocketException ex)
    {
        return new ProbeResult(probe.Protocol, probe.Address, true, $"Socket 错误码: {(int)ex.SocketErrorCode} ({ex.SocketErrorCode})。", ex.SocketErrorCode);
    }
    catch (Exception ex)
    {
        return new ProbeResult(probe.Protocol, probe.Address, true, $"异常: {ex.Message}", null);
    }
}
```

现在把这段代码拆开来看。

### Socket 创建与 ExclusiveAddressUse

```csharp
using var socket = new Socket(AddressFamily.InterNetwork, probe.SocketType, probe.ProtocolType)
{
    ExclusiveAddressUse = true
};
```

这里创建的是 IPv4 的 Socket，创建时直接通过属性初始化器设置了 `ExclusiveAddressUse = true`。

为什么要专门设置这个属性？因为如果不设置，`ReuseAddress` 在上层默认可能是开启的，这会导致你在 `Bind` 的时候即使有其他进程在用同一个端口，也可能不报错——但端口并没有真的空闲，后续 `Listen` 或通信时才会暴露问题。设成 `true` 就是在告诉操作系统：这个端口我要独占，别让其他进程通过地址复用来分享。

还有一个细节：`ExclusiveAddressUse` 必须在 `Bind` 之前设。一旦 `Bind` 执行了，端口归属就已经确定，此时再改这个属性就会抛异常。属性初始化器的写法（在 `new` 的同时把值赋好）正好保证了设置发生在 `Bind` 之前，顺序不会错。

至于 `probe.SocketType` 和 `probe.ProtocolType` 这两个参数，它们的值是由外部传入的探测组合决定的——TCP 对应 `SocketType.Stream` + `ProtocolType.Tcp`，UDP 对应 `SocketType.Dgram` + `ProtocolType.Udp`。这部分在后面"四项探测"一节会详细展开。

### Bind：真正的检测动作

```csharp
socket.Bind(new IPEndPoint(probe.Address, port));
```

这一行就是整个检测的核心。它将 Socket 绑定到指定的地址和端口上。绑定成功，说明这个地址+端口的组合目前是空闲的。绑定失败，`SocketException` 会被抛出，进入下面的 catch 分支。

`probe.Address` 可能是 `IPAddress.Loopback`（127.0.0.1）或 `IPAddress.Any`（0.0.0.0），`port` 是用户指定的端口号。这一行没有任何黑盒猜测——它就是真枪实弹地去占了再说。

### TCP 需要多一步 Listen

```csharp
if (probe.NeedsListen)
{
    socket.Listen(1);
}
```

TCP 的 Socket 在 `Bind` 之后，还需要调用 `Listen` 才能真正进入监听状态。UDP 是无连接的协议，`Bind` 就足够了，不需要 `Listen`。

如果对 TCP 只做 `Bind` 不做 `Listen`，虽然绑定成功了，但严格来说还没有进入"监听"状态——这会导致一些边界情况被漏掉。比如某些安全软件可能会在 `Listen` 阶段拦截端口，而 `Bind` 阶段放行。因此对于 TCP 探测，必须把 `Bind` + `Listen` 都执行一遍，检测结果才可靠。

`probe.NeedsListen` 由外部传入的探测参数决定——TCP 探测传入 `true`，UDP 探测传入 `false`。

### 保留 SocketErrorCode

```csharp
catch (SocketException ex)
{
    return new ProbeResult(probe.Protocol, probe.Address, true,
        $"Socket 错误码: {(int)ex.SocketErrorCode} ({ex.SocketErrorCode})。", ex.SocketErrorCode);
}
```

这里不只是简单地吃掉异常然后报告"失败"，而是把 `SocketErrorCode` 完整保留下来。

为什么要保留错误码？因为后续的分析需要区分不同的失败原因。`SocketError.AddressAlreadyInUse` 表示端口被其他进程占用，`SocketError.AccessDenied` 表示被系统或安全策略拒绝。这两种情况的排查方向完全不同——前者去查进程列表就能找到占用者，后者则要去查系统排除端口范围、HTTP.sys 配置、安全软件策略等。"智能分析"一节会详细说这部分逻辑。

`ProbeResult` 是一个 record 类型，承载了单次探测的完整结果：

```csharp
private sealed record ProbeResult(string Protocol, IPAddress Address, bool IsOccupied, string Message, SocketError? SocketErrorCode);
```

## 为什么要同时探测四种组合

`TryListen` 每次只探测一个"协议+地址"的组合，但实际使用时，调用方会循环四个不同的组合去探测同一个端口。这四个组合定义如下：

```csharp
private static readonly PortProbe[] Probes =
[
    new("TCP", IPAddress.Loopback, SocketType.Stream, ProtocolType.Tcp, NeedsListen: true),
    new("TCP", IPAddress.Any, SocketType.Stream, ProtocolType.Tcp, NeedsListen: true),
    new("UDP", IPAddress.Loopback, SocketType.Dgram, ProtocolType.Udp, NeedsListen: false),
    new("UDP", IPAddress.Any, SocketType.Dgram, ProtocolType.Udp, NeedsListen: false),
];
```

为什么是四个？因为 TCP 和 UDP 的端口命名空间是独立的——TCP 8080 被占用不影响 UDP 8080 的使用。而 127.0.0.1 和 0.0.0.0 的绑定也各有各的占用状态——某个程序可能只监听了 127.0.0.1:8080，此时 0.0.0.0:8080 依然空闲。把这四个组合全部测一遍，才能得到一个端口在"所有常见监听场景"下的完整可用性画像。

`PortProbe` 的定义也很简洁：

```csharp
private sealed record PortProbe(string Protocol, IPAddress Address, SocketType SocketType, ProtocolType ProtocolType, bool NeedsListen);
```

`NeedsListen` 字段标记了是否需要调用 `Listen`，把 TCP 和 UDP 的行为差异封装在数据中。

### 短路逻辑

四个探测的调用者是 `CanListenOnAllProbes` 方法：

```csharp
private static bool CanListenOnAllProbes(int port)
{
    foreach (var probe in Probes)
    {
        if (TryListen(probe, port).IsOccupied)
        {
            return false;
        }
    }

    return true;
}
```

这里用了短路逻辑：只要第一个探测失败就立刻返回 false，不再执行后续探测。这样做有两个好处：一是节省时间，四个探测全部执行需要四次系统调用，如果一个端口连第一个探测都过不了，没必要继续试；二是避免在已经确认被占用的端口上反复创建和销毁 Socket 带来的资源开销。

## 查端口占用者：Windows API 深度调用

探测到端口被占用后，还需要知道是谁占的。这就要查 Windows 的端口监听表，用到 `GetExtendedTcpTable` 和 `GetExtendedUdpTable` 这两个 Win32 API。

### P/Invoke 声明

```csharp
[DllImport("iphlpapi.dll", SetLastError = true)]
private static extern uint GetExtendedTcpTable(IntPtr tcpTable, ref int size, bool order,
    int ipVersion, TcpTableClass tableClass, uint reserved);
```

`iphlpapi.dll` 是 Windows IP Helper API 所在的 DLL。`TcpTableClass` 的取值为 `TcpTableOwnerPidListener = 3`，指定获取包含 PID 和监听状态信息的扩展 TCP 表——普通的 TCP 表只有基本连接信息，没有 PID。

### 两次调用模式

查询 TCP 表时，用的是 Windows API 中典型的两次调用模式：

```csharp
var size = 0;
_ = GetExtendedTcpTable(IntPtr.Zero, ref size, true, AfInet,
    TcpTableClass.TcpTableOwnerPidListener, 0);
```

第一次调用，`tcpTable` 传 `IntPtr.Zero`。API 发现缓冲区是空指针，就不填数据，而是在 `size` 中返回"我需要的缓冲区大小"。返回值此时是 `ERROR_INSUFFICIENT_BUFFER`，我们用 `_` 丢弃它，只关心 `size`。

拿到 `size` 后，分配非托管内存：

```csharp
buffer = Marshal.AllocHGlobal(size);
var result = GetExtendedTcpTable(buffer, ref size, true, AfInet,
    TcpTableClass.TcpTableOwnerPidListener, 0);
```

第二次调用传入了分配好的缓冲区，API 把数据填进去。如果 `result` 不为 0，说明出现了真正的错误，直接 `yield break` 返回空结果。

为什么要用 `Marshal.AllocHGlobal` 而不是 `new byte[]`？因为 Win32 API 要求内存块必须在非托管堆上，且不会被 GC 移动。用托管数组的话，GC 随时可能搬走数组，导致 API 写入错误的内存地址。`AllocHGlobal` 分配的内存在非托管堆上，地址固定，GC 管不着。

### 解析返回的内存布局

API 返回的缓冲区布局是：前 4 字节是一个 `uint`，表示条目数量，紧接着是结构体数组。

```csharp
var rowCount = Marshal.ReadInt32(buffer);
var rowPointer = IntPtr.Add(buffer, sizeof(int));
var rowSize = Marshal.SizeOf<MibTcpRowOwnerPid>();
```

`Marshal.ReadInt32(buffer)` 从缓冲区起始位置读出行数。`IntPtr.Add(buffer, sizeof(int))` 把指针偏移 4 字节，跳过行数计数器，指向第一条结构体的起始地址。`Marshal.SizeOf<MibTcpRowOwnerPid>()` 获取结构体大小，用于在循环中逐行偏移。

逐行遍历：

```csharp
for (var i = 0; i < rowCount; i++)
{
    var row = Marshal.PtrToStructure<MibTcpRowOwnerPid>(rowPointer);
    if (GetPort(row.LocalPort) == port)
    {
        yield return CreatePortOwner("TCP", row.LocalAddr, port, unchecked((int)row.OwningPid));
    }
    rowPointer = IntPtr.Add(rowPointer, rowSize);
}
```

`Marshal.PtrToStructure<MibTcpRowOwnerPid>(rowPointer)` 把非托管内存中的字节序列解释为 `MibTcpRowOwnerPid` 结构体。然后比较 `LocalPort` 是否匹配目标端口，匹配了就用 `yield return` 返回占用者信息。

最后必须释放内存：

```csharp
finally
{
    if (buffer != IntPtr.Zero)
    {
        Marshal.FreeHGlobal(buffer);
    }
}
```

`finally` 确保即使中间抛了异常，内存也不会泄漏。UDP 版本的 `GetUdpOwners` 逻辑几乎一样，只是 API、结构体和 `UdpTableClass` 不同。

### MibTcpRowOwnerPid 结构体

这个结构体的字段布局必须与 Windows API 的定义严格一致：

```csharp
[StructLayout(LayoutKind.Sequential)]
private struct MibTcpRowOwnerPid
{
    public uint State;
    public uint LocalAddr;
    public uint LocalPort;
    public uint RemoteAddr;
    public uint RemotePort;
    public uint OwningPid;
}
```

`StructLayoutKind.Sequential` 保证字段在内存中按声明顺序排列，CLR 不会做额外的填充或重排。字段类型必须用 `uint` 而不是 `int`，因为 Windows API 使用的是无符号整数。

UDP 版本更简单，因为没有连接状态和远程地址字段：

```csharp
[StructLayout(LayoutKind.Sequential)]
private struct MibUdpRowOwnerPid
{
    public uint LocalAddr;
    public uint LocalPort;
    public uint OwningPid;
}
```

### 网络字节序的端口号转换

API 返回的 `LocalPort` 是网络字节序（大端），而我们代码里用的是主机字节序（小端），需要转换：

```csharp
private static int GetPort(uint rawPort)
{
    var bytes = BitConverter.GetBytes(rawPort);
    return (bytes[0] << 8) + bytes[1];
}
```

`BitConverter.GetBytes` 把 `uint` 按小端拆成 4 个字节。对于网络字节序的值，`bytes[0]` 是大端的高位字节，`bytes[1]` 是大端的低位字节。`(bytes[0] << 8) + bytes[1]` 组合出来就是主机字节序的端口号。这等价于 `bytes[1] * 256 + bytes[0]`，也可以直接用 `IPAddress.NetworkToHostOrder`。

### 从 PID 拿到进程名

```csharp
private static string GetProcessName(int processId)
{
    try
    {
        using var process = Process.GetProcessById(processId);
        return process.ProcessName;
    }
    catch
    {
        return "未知进程";
    }
}
```

这里需要 try-catch 的原因是：进程可能已经退出了，或者当前用户没有权限访问该进程（系统进程、管理员进程）。这种情况下 `GetProcessById` 会抛异常，我们兜底返回"未知进程"，不影响整体检测流程。

### IP 地址的还原

API 返回的 `LocalAddr` 也是网络字节序的 `uint`，但 .NET 的 `IPAddress` 构造函数接受的是网络字节序的字节数组：

```csharp
private static PortOwner CreatePortOwner(string protocol, uint address, int port, int processId)
{
    var bytes = BitConverter.GetBytes(address);
    var ipAddress = new IPAddress(bytes);
    return new PortOwner(protocol, ipAddress, port, processId, GetProcessName(processId));
}
```

`BitConverter.GetBytes` 在主机字节序的机器上输出的是小端字节序，但对于网络字节序的 `uint` 来说，`BitConverter.GetBytes` 输出的第 0 个字节恰好是大端的最低位，这正好对应 `IPAddress` 构造函数要求的网络字节序排列。所以虽然绕了一下，但结果是正确的。

## 读取系统排除端口范围

Windows 有一个容易被忽略的端口占用机制：排除端口范围。这些端口被系统保留给 Hyper-V、WSL、某些 VPN 客户端等使用。它的特点是：端口表里查不到，也没有你能看到的进程在监听，但就是绑不上去，返回 `AccessDenied`。

获取这些范围用 `netsh` 命令：

```csharp
var output = RunProcessAndReadOutput("netsh",
    $"int ipv4 show excludedportrange protocol=tcp");
```

这个命令的输出格式类似：

```
协议 tcp 端口排除范围

开始端口    结束端口
--------    --------
5357        5357
50000       50059
```

需要从这段文本中提取数字。用正则表达式来解析：

```csharp
var matches = Regex.Matches(output, @"^\s*(\d+)\s+(\d+)\s*$", RegexOptions.Multiline);
foreach (Match match in matches)
{
    if (!int.TryParse(match.Groups[1].Value, out var start) ||
        !int.TryParse(match.Groups[2].Value, out var end))
    {
        continue;
    }
    ranges.Add(new PortRange(start, end));
}
```

正则 `^\s*(\d+)\s+(\d+)\s*$` 配合 `RegexOptions.Multiline`，精确匹配每行中"两个数字中间用空白隔开"的模式。在 Multiline 模式下，`^` 和 `$` 匹配的是行的首尾，不是整个字符串的首尾，所以一行只会命中一个端口范围。这避免了误匹配到中文表头或者空行。

`RunProcessAndReadOutput` 封装了进程启动：

```csharp
process.StartInfo = new ProcessStartInfo(fileName, arguments)
{
    UseShellExecute = false,
    RedirectStandardOutput = true,
    RedirectStandardError = true,
    CreateNoWindow = true
};
```

`UseShellExecute = false` 是关键。当它为 `true` 时，`RedirectStandardOutput` 会失效——因为 ShellExecute 模式走的是 shell 路径，标准输出不归 .NET 的 Process 组件管。`CreateNoWindow = true` 防止弹命令行黑窗口。另外还设了 `WaitForExit(5000)` 限时等待，避免 netsh 在特殊情况下卡住。

TCP 的排除端口范围和 UDP 的是分开管理的，所以在搜索可用端口时同时检查两方——只要命中了 TCP 或 UDP 任意一方的排除范围，这个端口就会被跳过。

## 自动查找可用端口的搜索策略

有了上面三项能力——真枪实弹的监听检测、系统端口表查询、排除端口范围读取——自动查找可用端口就水到渠成了。

### 搜索区间的优先级设计

```csharp
private static IEnumerable<PortRange> GetPreferredSearchRanges()
{
    yield return new PortRange(10000, 60000);
    yield return new PortRange(1024, 9999);
    yield return new PortRange(60001, 65535);
}
```

为什么首选 10000-60000？这个范围避开了两个容易冲突的区域：1024 以下是特权端口，需要管理员权限才能绑定；1024-9999 是很多应用程序和开发工具的常用范围，冲突概率最高。而 60001-65535 可能与 Windows 的动态端口分配范围重叠，通常从 49152 开始，所以放在最后扫描。

### 逐端口扫描

```csharp
for (var port = range.Start; port <= range.End; port++)
{
    if (ContainsPort(tcpExcludedRanges, port) || ContainsPort(udpExcludedRanges, port))
    {
        continue;
    }

    checkedCount++;
    if (checkedCount % SearchProgressInterval == 0)
    {
        progress.Report($"已扫描 {checkedCount} 个候选端口，当前检查到 {port}。");
    }

    if (!CanListenOnAllProbes(port))
    {
        continue;
    }

    progress.Report($"找到可用于 TCP/UDP 与 127.0.0.1/0.0.0.0 的可用端口：{port}。");
    return new PortSearchResult(port, []);
}
```

每个候选端口先检查是否落在 TCP 或 UDP 的排除范围内，命中就直接跳过——这种端口连试都不用试，系统已经明确说"这个不给你用"。

`SearchProgressInterval` 是常量 500，每扫描 500 个端口才汇报一次进度。如果每个端口都调用 `progress.Report`，字符串拼接和 UI 线程的消息投递会成为新的瓶颈，反而让扫描变慢。

当找到一个四项探测全部通过的端口后，立即返回。这里不需要找"最优"端口，只需要找"第一个可用"的——就近原则最省时间。

### 找不到时的退路

三个区间全部扫完还没找到，就生成分析建议：

```csharp
var analysis = new List<string>
{
    "在常用扫描区间内未找到同时满足四项监听条件的端口。",
    "如果几乎所有端口都报 AccessDenied，优先检查 Windows 排除端口范围、Hyper-V/HNS、HTTP.sys、VPN/代理或安全软件。"
};
return new PortSearchResult(null, analysis);
```

这里的分析建议和 `AnalyzePortIssues` 中的知识体系是呼应的——它们基于同一个认知：AccessDenied 且没有进程占用记录时，大概率是内核组件或系统策略导致。

## 智能分析：不只是"被占了"

检测端口时，如果只输出"可用"或"被占用"，很多时候用户还是无从下手。所以工具在检测结束后会生成分析建议，分析逻辑在 `AnalyzePortIssues` 方法中。

### 第一层：全部通过

```csharp
if (probeResults.All(result => !result.IsOccupied))
{
    analysis.Add("四项监听均成功，此端口可直接用于本机监听。");
    return analysis;
}
```

四项探测全部成功，说明这个端口完全空闲。直接给出结论并返回，不需要后续分析。

### 第二层：AddressAlreadyInUse

```csharp
if (probeResults.Any(result => result.SocketErrorCode == SocketError.AddressAlreadyInUse))
{
    analysis.Add("出现 AddressAlreadyInUse，说明该端口已经被现有监听直接占用。");
}
```

这是最常见也是最好处理的占用形式。此时结合前面查询到的进程信息（PID 和进程名），用户可以直接找到是哪个程序占用的。

### 第三层：AccessDenied 的分层排查

`AccessDenied` 是更复杂的情况。它表示系统拒绝了这个绑定请求，不一定是普通进程冲突。

第一步，检查是否命中了系统排除端口范围：

```csharp
var tcpExcludedRange = FindContainingRange(GetExcludedPortRanges("tcp"), port);
if (tcpExcludedRange is not null)
{
    analysis.Add($"该端口命中 TCP 排除端口范围 {tcpExcludedRange.Start}-{tcpExcludedRange.End}，系统可能直接拒绝监听。");
}
```

`FindContainingRange` 的实现非常简洁：

```csharp
private static PortRange? FindContainingRange(IReadOnlyList<PortRange> ranges, int port)
{
    return ranges.FirstOrDefault(range => range.Start <= port && port <= range.End);
}
```

用 `FirstOrDefault` 配合范围判断，一行就表达了"是否存在包含该端口的范围"的语义，而 `ContainsPort` 进一步复用了它。

第二步，检查是否存在"环回失败但全局成功"的不对称现象：

```csharp
var loopbackDeniedProtocols = probeResults
    .Where(result => result.Address.Equals(IPAddress.Loopback) && result.SocketErrorCode == SocketError.AccessDenied)
    .Select(result => result.Protocol)
    .Distinct()
    .ToList();

if (loopbackDeniedProtocols.Any(protocol => probeResults.Any(result =>
    result.Protocol == protocol && result.Address.Equals(IPAddress.Any) && !result.IsOccupied)))
{
    analysis.Add("同协议下 127.0.0.1 失败而 0.0.0.0 成功，更像是环回地址被代理、安全软件、端口转发组件或系统策略限制。");
}
```

这里的判断逻辑是：同一个协议下，127.0.0.1 被拒但 0.0.0.0 成功。如果两个都被拒，那可能是全局性的端口占用或系统策略；只有环回地址被拒，则更像是代理软件（如 Clash、ssr）或 VPN 在环回地址上的流量拦截。

第三步，结合监听表给出建议：

```csharp
if (owners.Count == 0)
{
    analysis.Add("监听表未返回对应进程时，常见原因是 Windows 端口保留、HTTP.sys、Hyper-V/HNS、VPN/代理或安全软件。");
}

if (!IsRunningAsAdministrator())
{
    analysis.Add("当前程序未以管理员身份运行，可尝试提升权限后再次验证，以排除本机安全策略影响。");
}
```

监听表查不到进程但仍然无法绑定，说明不是普通用户态进程在占用。这在 Windows 上很常见——HTTP.sys 可以通过 `netsh http show servicestate` 看到它占用了哪些端口，但不会出现在 TCP 监听表中。Hyper-V 和 HNS 同理。

权限提示是因为 `AccessDenied` 在非管理员权限下更容易遇到。

### 兜底

```csharp
if (analysis.Count == 0)
{
    analysis.Add("检测失败，但未匹配到典型问题模式，请结合详细日志进一步确认。");
}
```

用 `analysis.Count == 0` 作为兜底条件——前面所有条件都没有命中，说明遇到了预料之外的情况，给出一条通用提示。

## WPF UI 设计

工具界面上层是标题区域，中间是输入区（端口号输入框 + "开始检测"按钮 + "自动找可用端口"按钮），底层是深色背景的日志输出区，使用 Consolas 等宽字体展示时间戳和实时检测日志。

所有耗时操作——端口检测、自动查找——都放在 `Task.Run` 中执行，通过 `IProgress<string>` 汇报进度到 UI 线程：

```csharp
private async Task ExecuteBusyActionAsync(string startMessage, Func<IProgress<string>, Task> action)
{
    SetBusyState(true);
    LogTextBox.Clear();
    AppendLog(startMessage);

    try
    {
        var progress = new Progress<string>(AppendLog);
        await action(progress);
    }
    finally
    {
        SetBusyState(false);
    }
}
```

`Progress<string>` 的构造函数传入 `AppendLog` 回调，内部会自动把 `Report` 调用封送到创建它的 UI 线程，所以日志输出天然是线程安全的。`SetBusyState` 在操作期间禁用按钮，防止用户重复点击。

## 开源

本工具全部代码开源在 GitHub 和 Gitee 上，项目基于 .NET 10 和 WPF 构建。通过本文的讲解，你也可以将核心的 `TryListen` 方法、`GetExtendedTcpTable`/`GetExtendedUdpTable` P/Invoke 调用以及 `netsh` 排除端口范围查询这三部分逻辑抽取到自己的项目中，实现自己的端口检测能力。

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1ee6c50c85be684eba0c9e8ffa00bc47984165f0/Workbench/NetPortFinder) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1ee6c50c85be684eba0c9e8ffa00bc47984165f0/Workbench/NetPortFinder) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1ee6c50c85be684eba0c9e8ffa00bc47984165f0
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1ee6c50c85be684eba0c9e8ffa00bc47984165f0
```

获取代码之后，进入 Workbench/NetPortFinder 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
