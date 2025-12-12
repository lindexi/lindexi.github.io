
# dotnet 制作一个简单的自动更新系统日期时间工具

本文和大家介绍我制作的一个简单的开机自启的自动更新系统日期时间工具

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

整体技术非常简单：

- 采用 .NET 10 + AOT 方式制作。运行占用物理内存低至几百 KB 大小
- 采用 [dotnet6 C# 一个国内还能用的 NTP 时间校准客户端的实现](https://blog.lindexi.com/post/dotnet6-C-%E4%B8%80%E4%B8%AA%E5%9B%BD%E5%86%85%E8%BF%98%E8%83%BD%E7%94%A8%E7%9A%84-NTP-%E6%97%B6%E9%97%B4%E6%A0%A1%E5%87%86%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84%E5%AE%9E%E7%8E%B0.html ) 博客提供的方法，使用国内的腾讯（ntp.tencent.com）、阿里（ntp.aliyun.com）等国内网络时间服务同步时间
- 调用系统的 kernel32.SetLocalTime 设置时间
- 使用 `Microsoft.Extensions.Hosting.WindowsServices` 库和 sc 命令注册为开机启动的服务
- 添加 `app.manifest` 设置管理员权限

下载地址： <https://github.com/lindexi/UWP/releases/tag/AutoUpdateSystemTime_1.0.2>

全部代码都在一个文件完成，代码如下

```csharp
using System.Buffers;
using System.Diagnostics;
using System.Net;
using System.Net.Sockets;
using System.Runtime.InteropServices;
using System.Threading;

using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;

HostApplicationBuilder builder = Host.CreateApplicationBuilder(args);

builder.Services.AddWindowsService(options => { options.ServiceName = "AutoUpdateSystemTime"; });

builder.Services.AddHostedService<WindowsBackgroundService>();
builder.Services.AddLogging(loggingBuilder => loggingBuilder.ClearProviders());

var host = builder.Build();

Console.WriteLine("本程序将自动从腾讯（ntp.tencent.com）、阿里（ntp.aliyun.com）等国内网络时间服务同步时间");
Console.WriteLine($"本程序开机自启，可有效解决主板小电池没电导致日期时间被重置的问题");
Console.WriteLine();
Console.WriteLine($"本程序代码完全开源在 GitHub 上: https://github.com/lindexi/UWP/tree/master/app/Tool/自动同步系统时间/AutoUpdateSystemTime");
Console.WriteLine();

if (OperatingSystem.IsWindows())
{
    Process.Start("sc.exe",
    [
        "create",
        "UpdateTimeServer",
        $"binPath=", $"\"{Environment.ProcessPath!}\"",
        "start=", "auto",
        "displayname=", "自动更新时间服务"
    ]);

    Console.WriteLine("已创建 Windows 服务：UpdateTimeServer （自动更新时间服务），启动类型为自动开机自启。如需删除该服务，请运行命令：sc delete UpdateTimeServer，或直接删除本可执行 exe 文件");
    Console.WriteLine();
}

Console.WriteLine($"开始自动更新时间");
Console.WriteLine();
await host.RunAsync();

Console.Read();

public class WindowsBackgroundService : BackgroundService
{
    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var currentProcess = Process.GetCurrentProcess();
        _ = Task.Run(async () =>
        {
            while (true)
            {
                var networkTime = await NtpClient.GetChineseNetworkTime();
                if (networkTime != null)
                {
                    TimeSpan gapInTime = DateTimeOffset.Now - networkTime.Value;
                    var absSeconds = Math.Abs(gapInTime.TotalSeconds);
                    // 时间差距大于 5 秒才进行校准
                    if (absSeconds > 5)
                    {
                        Console.WriteLine($"正在将系统时间从 {DateTimeOffset.Now:yyyy-MM-dd HH:mm:ss} 同步到网络时间 {networkTime.Value:yyyy-MM-dd HH:mm:ss} ......");
                        SetNtpTime(networkTime.Value);
                    }
                }

                EmptyWorkingSet(currentProcess.Handle);
                await Task.Delay(TimeSpan.FromMinutes(1));
            }
        });

        return Task.CompletedTask;
    }

    /// <summary>
    /// 为系统设置输入的时间
    /// </summary>
    public static bool SetNtpTime(DateTimeOffset time)
    {
        try
        {
            var localTime = time.LocalDateTime;

            var result = SetLocalTime(new SYSTEMTIME
            {
                wYear = (ushort) localTime.Year,
                wMonth = (ushort) localTime.Month,
                wDayOfWeek = (ushort) localTime.DayOfWeek,
                wDay = (ushort) localTime.Day,
                wHour = (ushort) localTime.Hour,
                wMinute = (ushort) localTime.Minute,
                wSecond = (ushort) localTime.Second,
                wMilliseconds = (ushort) localTime.Millisecond
            });

            return result;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"设置系统时间失败 {ex}");
            return false;
        }
    }

    /// <summary>
    /// <para>
    /// Sets the current local time and date.
    /// </para>
    /// <para>
    /// From: <see href="https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-setlocaltime"/>
    /// </para>
    /// </summary>
    /// <param name="lpSystemTime">
    /// A pointer to a <see cref="SYSTEMTIME"/> structure that contains the new local date and time.
    /// The <see cref="SYSTEMTIME.wDayOfWeek"/> member of the <see cref="SYSTEMTIME"/> structure is ignored.
    /// </param>
    /// <returns>
    /// If the function succeeds, the return value is <see cref="TRUE"/>.
    /// If the function fails, the return value is <see cref="FALSE"/>.
    /// To get extended error information, call <see cref="GetLastError"/>.
    /// </returns>
    /// <remarks>.
    /// The calling process must have the <see cref="SE_SYSTEMTIME_NAME"/> privilege. This privilege is disabled by default.
    /// The <see cref="SetLocalTime"/> function enables the <see cref="SE_SYSTEMTIME_NAME"/> privilege
    /// before changing the local time and disables the privilege before returning.
    /// For more information, see Running with Special Privileges.
    /// The system uses UTC internally.
    /// Therefore, when you call <see cref="SetLocalTime"/>, the system uses the current time zone information
    /// to perform the conversion, including the daylight saving time setting.
    /// Note that the system uses the daylight saving time setting of the current time, not the new time you are setting.
    /// Therefore, to ensure the correct result, call <see cref="SetLocalTime"/> a second time,
    /// now that the first call has updated the daylight saving time setting.
    /// </remarks>
    [DllImport("kernel32.dll", CharSet = CharSet.Unicode, EntryPoint = "SetLocalTime", ExactSpelling = true,
        SetLastError = true)]
    public static extern bool SetLocalTime([In] in SYSTEMTIME lpSystemTime);

    [DllImport("psapi")]
    public static extern bool EmptyWorkingSet(IntPtr hProcess);
}

/// <summary>
/// <para>
/// Specifies a date and time, using individual members for the month, day, year, weekday, hour, minute, second, and millisecond.
/// The time is either in coordinated universal time (UTC) or local time, depending on the function that is being called.
/// </para>
/// <para>
/// From: <see href="https://learn.microsoft.com/en-us/windows/win32/api/minwinbase/ns-minwinbase-systemtime"/>
/// </para>
/// </summary>
/// <remarks>
/// The <see cref="SYSTEMTIME"/> does not check to see if the date represented is a real and valid date.
/// When working with this API, you should ensure its validity, especially in leap reat scenarios.
/// See leap day readiness for more information.
/// It is not recommended that you add and subtract values from the <see cref="SYSTEMTIME"/> structure to obtain relative times.
/// Instead, you should:
/// Convert the <see cref="SYSTEMTIME"/> structure to a <see cref="FILETIME"/> structure.
/// Copy the resulting <see cref="FILETIME"/> structure to a <see cref="ULARGE_INTEGER"/> structure.
/// Use normal 64-bit arithmetic on the <see cref="ULARGE_INTEGER"/> value.
/// The system can periodically refresh the time by synchronizing with a time source.
/// Because the system time can be adjusted either forward or backward, do not compare system time readings to determine elapsed time.
/// Instead, use one of the methods described in Windows Time.
/// </remarks>
[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
public struct SYSTEMTIME
{
    /// <summary>
    /// The year. The valid values for this member are 1601 through 30827.
    /// </summary>
    public WORD wYear;

    /// <summary>
    /// The month. This member can be one of the following values.
    /// 1 January
    /// 2 February
    /// 3 March
    /// 4 April
    /// 5 May
    /// 6 June
    /// 7 July
    /// 8 August
    /// 9 September
    /// 10 October
    /// 11 November
    /// 12 December
    /// </summary>
    public WORD wMonth;

    /// <summary>
    /// The day of the week. This member can be one of the following values.
    /// 0 Sunday
    /// 1 Monday
    /// 2 Tuesday
    /// 3 Wednesday
    /// 4 Thursday
    /// 5 Friday
    /// 6 Saturday
    /// </summary>
    public WORD wDayOfWeek;

    /// <summary>
    /// The day of the month. The valid values for this member are 1 through 31.
    /// </summary>
    public WORD wDay;

    /// <summary>
    /// The hour. The valid values for this member are 0 through 23.
    /// </summary>
    public WORD wHour;

    /// <summary>
    /// The minute. The valid values for this member are 0 through 59.
    /// </summary>
    public WORD wMinute;

    /// <summary>
    /// The second. The valid values for this member are 0 through 59.
    /// </summary>
    public WORD wSecond;

    /// <summary>
    /// The millisecond. The valid values for this member are 0 through 999.
    /// </summary>
    public WORD wMilliseconds;
}

/// <summary>
/// <para>
/// A <see cref="WORD"/> is a 16-bit unsigned integer (range: 0 through 65535 decimal). Because a WORD is unsigned,
/// its first bit (Most Significant Bit (MSB)) is not reserved for signing.
/// </para>
/// <para>
/// From: <see href="https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/f8573df3-a44a-4a50-b070-ac4c3aa78e3c"/>
/// </para>
/// </summary>
[StructLayout(LayoutKind.Explicit, Size = 2)]
public struct WORD
{
    [FieldOffset(0)] private ushort _value;

    /// <inheritdoc/>
    public override string ToString() => _value.ToString();

    /// <summary>
    /// 
    /// </summary>
    /// <param name="val"></param>
    public static implicit operator ushort(WORD val) => val._value;

    /// <summary>
    /// 
    /// </summary>
    /// <param name="val"></param>
    public static implicit operator WORD(ushort val) => new WORD { _value = val };

    /// <summary>
    /// 
    /// </summary>
    /// <param name="val"></param>
    public static explicit operator short(WORD val) => unchecked((short) val._value);

    /// <summary>
    /// 
    /// </summary>
    /// <param name="val"></param>
    public static explicit operator WORD(short val) => new WORD { _value = unchecked((ushort) val) };

    /// <summary>
    /// 
    /// </summary>
    /// <param name="val"></param>
    public static implicit operator uint(WORD val) => val._value;

    /// <summary>
    /// 
    /// </summary>
    /// <param name="val"></param>
    public static implicit operator int(WORD val) => val._value;
}

// [dotnet6 C# 一个国内还能用的 NTP 时间校准客户端的实现](https://blog.lindexi.com/post/dotnet6-C-%E4%B8%80%E4%B8%AA%E5%9B%BD%E5%86%85%E8%BF%98%E8%83%BD%E7%94%A8%E7%9A%84-NTP-%E6%97%B6%E9%97%B4%E6%A0%A1%E5%87%86%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84%E5%AE%9E%E7%8E%B0.html ) 
// https://github.com/michaelschwarz/NETMF-Toolkit/blob/095b01679945c3f518dd52082eca78bbaff9811f/NTP/NtpClient.cs
public static class NtpClient
{
    /// <summary>
    /// 国内的授时服务提供的网络时间。默认返回北京时区的时间。如需转换为本机时区时间，请使用 <code> var dateTimeOffset = NtpClient.GetChineseNetworkTime();var 本机时区时间 = dateTimeOffset.LocalDateTime;</code> 转换。本机时区时间和北京时间的差别是，本机系统时区可能被设置为非北京时间，当本机系统时区设置为北京时间，则本机时区时间和北京时间相同
    /// </summary>
    /// <remarks>实现方法是去询问腾讯和阿里的授时服务器</remarks>
    /// <returns>返回空表示没有能够获取到任何的时间，预计是网络错误了。返回北京时区的时间</returns>
    /// 本来想着异常对外抛出的，但是似乎抛出异常也没啥用
    public static async ValueTask<DateTimeOffset?> GetChineseNetworkTime()
    {
        // 感谢 [国内外常用公共NTP网络时间同步服务器地址_味辛的博客-CSDN博客_ntp服务器](https://blog.csdn.net/weixin_42588262/article/details/82501488 )
        var dateTimeOffset = await GetChineseNetworkTimeCore("ntp.tencent.com"); // 腾讯
        dateTimeOffset ??= await GetChineseNetworkTimeCore("ntp.aliyun.com"); // 阿里
        dateTimeOffset ??= await GetChineseNetworkTimeCore("cn.pool.ntp.org"); // 国家服务器
        dateTimeOffset ??= await GetChineseNetworkTimeCore("cn.ntp.org.cn"); // 中国授时
        dateTimeOffset ??= await GetChineseNetworkTimeCore("time.windows.com"); // time.windows.com 微软Windows自带
        // 203.107.6.88 是 ntp.aliyun.com 的 IP 地址之一，作为最后的兜底
        
        if (dateTimeOffset is null)
        {
            var ipEndPoint = new IPEndPoint(IPAddress.Parse("203.107.6.88"), 123);

            try
            {
                var cancellationTokenSource = new CancellationTokenSource();
                cancellationTokenSource.CancelAfter(TimeSpan.FromSeconds(15));

                dateTimeOffset = await GetNetworkUtcTime(ipEndPoint, cancellationTokenSource.Token);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"无法从 {ipEndPoint.Address} 获取时间 {ex}");
            }
        }

        if (dateTimeOffset is not null)
        {
            return dateTimeOffset.Value.ToOffset(TimeSpan.FromHours(8));
        }
        else
        {
            return null;
        }

        static async ValueTask<DateTimeOffset?> GetChineseNetworkTimeCore(string ntpServer)
        {
            var cancellationTokenSource = new CancellationTokenSource();
            try
            {
                var hostEntry = await Dns.GetHostEntryAsync(ntpServer);
                IPAddress[] addressList = hostEntry.AddressList;

                if (addressList.Length == 0)
                {
                    // 被投毒了？那就换其他一个吧
                    return null;
                }

                foreach (var address in addressList)
                {
                    try
                    {
                        var ipEndPoint = new IPEndPoint(address, 123);
                        cancellationTokenSource.CancelAfter(TimeSpan.FromSeconds(15));

                        return await GetNetworkUtcTime(ipEndPoint, cancellationTokenSource.Token);
                    }
                    catch (Exception ex)
                    {
                        Console.WriteLine($"无法从 {address} 获取时间 {ex}");
                    }

                    if (!cancellationTokenSource.TryReset())
                    {
                        cancellationTokenSource.Dispose();
                        cancellationTokenSource = new CancellationTokenSource();
                    }
                }
            }
            catch(Exception ex)
            {
                // 失败就失败
                // 本来想着异常对外抛出的，但是似乎抛出异常也没啥用
                Console.WriteLine($"无法从 {ntpServer} 获取时间 Exception: {ex}");
                Console.WriteLine();
            }
            finally
            {
                cancellationTokenSource.Dispose();
            }

            return null;
        }
    }

    /// <summary>
    /// Gets the current DateTime from time-a.nist.gov.
    /// </summary>
    /// <returns>A DateTime containing the current time.</returns>
    public static ValueTask<DateTimeOffset> GetNetworkUtcTime()
    {
        return GetNetworkUtcTime("time-a.nist.gov");
    }

    /// <summary>
    /// Gets the current DateTime from <paramref name="ntpServer"/>.
    /// </summary>
    /// <param name="ntpServer">The hostname of the NTP server.</param>
    /// <returns>A DateTime containing the current time.</returns>
    public static async ValueTask<DateTimeOffset> GetNetworkUtcTime(string ntpServer)
    {
        var hostEntry = await Dns.GetHostEntryAsync(ntpServer);
        IPAddress[] address = hostEntry.AddressList;

        if (address == null || address.Length == 0)
        {
            throw new ArgumentException($"Could not resolve ip address from '{ntpServer}'.", "ntpServer");
        }

        var ipEndPoint = new IPEndPoint(address[0], 123);

        return await GetNetworkUtcTime(ipEndPoint);
    }

    /// <summary>
    /// Gets the current DateTime form <paramref name="endPoint"/> IPEndPoint.
    /// </summary>
    /// <param name="endPoint">The IPEndPoint to connect to.</param>
    /// <param name="token"></param>
    /// <returns>A DateTime containing the current time.</returns>
    public static async ValueTask<DateTimeOffset> GetNetworkUtcTime(IPEndPoint endPoint,
        CancellationToken token = default)
    {
        using var socket = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);

        await socket.ConnectAsync(endPoint, token);

        const int length = 48;

        // 实现方法请参阅 RFC 2030 的内容
        var ntpData = ArrayPool<byte>.Shared.Rent(length);

        try
        {
            // 初始化数据
            ntpData[0] = 0x1B;
            for (int i = 1; i < length; i++)
            {
                ntpData[i] = 0;
            }

            await socket.SendAsync(ntpData.AsMemory(0, length), token);
            await socket.ReceiveAsync(ntpData.AsMemory(0, length), token);

            byte offsetTransmitTime = 40;
            ulong intPart = 0;
            ulong fractPart = 0;

            for (int i = 0; i <= 3; i++)
            {
                intPart = 256 * intPart + ntpData[offsetTransmitTime + i];
            }

            for (int i = 4; i <= 7; i++)
            {
                fractPart = 256 * fractPart + ntpData[offsetTransmitTime + i];
            }

            ulong milliseconds = (intPart * 1000 + (fractPart * 1000) / 0x100000000L);

            TimeSpan timeSpan = TimeSpan.FromMilliseconds(milliseconds);

            var dateTime = new DateTime(1900, 1, 1);
            dateTime += timeSpan;

            var dateTimeOffset = new DateTimeOffset(dateTime, TimeSpan.Zero);

            return dateTimeOffset;
        }
        finally
        {
            ArrayPool<byte>.Shared.Return(ntpData);
        }
    }
}
```

项目文件代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <ApplicationManifest>app.manifest</ApplicationManifest>
    <PublishAot>true</PublishAot>
    <PackageIcon>Icon.ico</PackageIcon>
    <ApplicationIcon>Icon.ico</ApplicationIcon>
    <Version>1.0.2</Version>

    <!-- [dotnet 9 已知问题 默认开启 CET 导致进程崩溃 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18700406 )  -->
    <CETCompat>false</CETCompat>

    <SupportedOSPlatformVersion>5.1</SupportedOSPlatformVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.Hosting.WindowsServices" Version="10.0.1" />
    <PackageReference Include="VC-LTL" Version="5.2.1" />
    <PackageReference Include="YY-Thunks" Version="1.1.7" />
  </ItemGroup>

</Project>
```

本文代码放在 [github](https://github.com/lindexi/UWP/tree/9fe650beba6596402c4c835a7a0d2fcd0021abc2/app/Tool/%E8%87%AA%E5%8A%A8%E5%90%8C%E6%AD%A5%E7%B3%BB%E7%BB%9F%E6%97%B6%E9%97%B4/AutoUpdateSystemTime) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 9fe650beba6596402c4c835a7a0d2fcd0021abc2
```

获取代码之后，进入 app/Tool/自动同步系统时间/AutoUpdateSystemTime 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。